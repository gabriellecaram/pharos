# IncidentFlow — Triagem e Escalonamento de Incidentes com IA

> Projeto de portfólio de Engenharia de Software com IA, no contexto de delivery (inspirado em operação tipo iFood). Usa apenas ferramentas gratuitas e locais.

## 1. Problema

Hoje: o cliente reclama → o atendente pergunta o que houve → abre o card manualmente → avisa alguém → ninguém confirma → o incidente fica parado.

Solução: um bot recebe o relato, a **IA faz a triagem** (classifica, extrai dados, detecta duplicidade, pede o que falta), **abre o chamado** e **notifica o plantão com escalonamento automático** até alguém confirmar o recebimento.

## 2. Fluxo principal

1. O cliente envia mensagem ao **bot do Telegram** → webhook → API → evento `MensagemRecebida` (RabbitMQ).
2. O **Worker de Triagem** (agente LLM com tools):
   - classifica a categoria (pedido errado, atraso, cobrança, item faltando, bug no app, outro) e a severidade (P1–P4);
   - extrai campos estruturados (nº do pedido, restaurante, valor, horário) em JSON validado por schema;
   - faz RAG sobre runbooks e incidentes passados → sugere o time responsável e **detecta duplicidade**;
   - se faltar informação essencial, **pergunta ao cliente** (máx. N turnos).
3. Abre o chamado via `ITicketProvider` (GitHub Issues ou Jira Cloud Free) e persiste no Postgres.
4. Notifica o 1º da escala do time com o link do chamado e o botão **"Confirmar recebimento"**.
5. Publica `VerificarConfirmacao` com atraso X (TTL + DLX):
   - confirmado → encerra o ciclo de escalonamento;
   - não confirmado → **escala para o próximo** e reagenda; ao fim da lista, alerta o líder.
6. **Feedback:** o responsável marca "classificação/time incorretos" → vira dado de avaliação.

## 3. Substituições gratuitas

| Real | Substituto | Abstração |
|---|---|---|
| Canal do cliente | Telegram Bot API | `IChatChannel` |
| Teams/Slack | Telegram (chats do plantão) | `INotificationChannel` |
| Jira | GitHub Issues (padrão) / Jira Cloud Free | `ITicketProvider` |
| Pessoas/plantão | Escala no Postgres + contas de teste | — |
| Clientes reais | Reclamações sintéticas geradas por LLM | — |
| LLM pago | Ollama local (padrão) / Gemini ou Groq free tier | `IChatClient` (Microsoft.Extensions.AI) |

## 4. Stack

- **Runtime:** .NET 10, ASP.NET Core Minimal APIs, Worker Service
- **IA:** Microsoft.Extensions.AI + OllamaSharp; modelos `qwen2.5:7b` (tools/JSON) e `nomic-embed-text` (embeddings)
- **Vetorial:** Qdrant — justificativa: open source, roda em Docker, filtros por payload (time, status, data) junto da similaridade, cliente .NET oficial, gRPC
- **Relacional:** PostgreSQL + EF Core (migrations)
- **Mensageria:** RabbitMQ (RabbitMQ.Client); escalonamento via TTL + Dead Letter Exchange
- **Resiliência:** Microsoft.Extensions.Http.Resilience (Polly): retry, circuit breaker, timeout
- **Validação:** FluentValidation
- **Observabilidade:** Serilog (JSON) + OpenTelemetry → Jaeger (traces), Prometheus + Grafana (métricas)
- **Testes:** xUnit, NSubstitute, Testcontainers (Postgres, RabbitMQ, Qdrant), WireMock.Net, Microsoft.Extensions.AI.Evaluation
- **Infra/CI:** Docker Compose, GitHub Actions, imagens no GHCR

## 5. Arquitetura

Clean Architecture com features em **Vertical Slice** na camada Application.

```
src/
  IncidentFlow.Domain/          entidades, value objects, regras (escalonamento, severidade)
  IncidentFlow.Application/     features (slices), portas (interfaces), DTOs
  IncidentFlow.Infrastructure/  EF Core, RabbitMQ, Qdrant, Ollama, Telegram, GitHub/Jira
  IncidentFlow.Api/             webhook, endpoints admin, auth, rate limit, ProblemDetails
  IncidentFlow.Worker/          consumers: triagem, ingestão, escalonamento
  IncidentFlow.Ingestion.Cli/   carga de runbooks/histórico
tests/  Unit/  Integration/  Evaluation/
eval/   golden-set.jsonl, relatórios
docs/   README, ARQUITETURA, IA, ADRs, RELATORIO
```

**Domínio:** `Incidente`, `MensagemCliente`, `Classificacao` (categoria, severidade, confiança, versão do prompt, modelo), `Chamado` (id e URL externos), `Time`, `MembroPlantao`, `EscalaPlantao`, `TentativaNotificacao`, `Feedback`, `DocumentoConhecimento`, `ExecucaoLlm` (tokens, latência, custo estimado, trace id).

**Eventos:** `MensagemRecebida`, `IncidenteTriado`, `ChamadoAberto`, `NotificacaoEnviada`, `VerificarConfirmacao`, `RecebimentoConfirmado`, `IncidenteEscalado`, `DocumentoIngerido`.

## 6. Pipeline de ingestão (RAG)

| Etapa | O que faz | Decisão a justificar |
|---|---|---|
| Ingestão | upload via API/CLI (MD, PDF, JSON de incidentes) → fila | assíncrono para não bloquear a API |
| Processamento | extração de texto, limpeza, deduplicação por hash | idempotência |
| Transformação | chunking por seção (~500 tokens, overlap 50) + metadados + embeddings | tamanho do chunk × precisão |
| Armazenamento | Qdrant (vetores + payload) + Postgres (metadados, status) | separar busca de fonte da verdade |

**Recuperação:** top-k com filtro por payload → limiar de similaridade → (opcional) reranking por LLM → contexto com citações.

## 7. Requisitos não funcionais

- **Segurança:** secret token do webhook do Telegram; JWT + roles (admin/plantonista) na API admin; rate limiting por chat/IP; secrets via `.env`/user-secrets (nunca no repositório); validação de entrada; **defesa contra prompt injection** (delimitação da entrada do usuário, saída estruturada, allowlist de tools, sem execução de ações destrutivas pelo LLM).
- **Observabilidade:** trace ponta a ponta (webhook → fila → LLM → ticket → notificação) com propagação de contexto no RabbitMQ; métricas: tempo até confirmação, taxa de escalonamento, latência/tokens do LLM, falhas de parsing, acurácia online via feedback; dashboard Grafana; erros correlacionados por `trace_id`.
- **Erros:** middleware global com ProblemDetails; DLQ para mensagens envenenadas; fallback para "triagem manual" se o LLM falhar.
- **LLMOps:** prompts versionados em arquivos (`prompts/triagem/v1.md`), versão gravada em cada `Classificacao`; troca de modelo por configuração.

## 8. Avaliação de qualidade da IA

- **Golden set:** ~150 reclamações sintéticas rotuladas (categoria, severidade, campos, time, duplicata).
- **Métricas:** acurácia/F1 por categoria, exatidão da severidade, F1 da extração por campo, precisão/recall de duplicidade, faithfulness/relevância das respostas RAG (LLM-as-judge com modelo diferente do avaliado), consistência (mesma entrada, N execuções).
- **Benchmark:** Ollama (2 modelos) × Gemini/Groq em qualidade, latência e custo.
- **CI:** LLM fake nos testes; avaliação completa local/manual com relatório versionado em `eval/reports/`.

## 9. Diferenciais cobertos

Function calling/tool use, agente com loop de esclarecimento, filas e eventos, processamento assíncrono, cache de embeddings, versionamento de prompts/modelos, avaliação automática, feedback loop, benchmark com experimentos reproduzíveis.

## 10. Entregáveis

Repositório público · `docker compose up` sobe tudo · CI verde · README + ARQUITETURA (diagramas C4/Mermaid) + IA + ADRs · relatório de avaliação · vídeo de demo (cliente reclama → chamado criado → 1º ignora → escala → trace no Jaeger).

## 11. Fora de escopo

Interface web para o cliente, multi-idioma, fine-tuning (possível extensão futura com LoRA), deploy em nuvem.

## Regras de implementação

- Nunca criar código sem testes.
- Sempre explicar a motivação da decisão arquitetural.
- Sempre respeitar Clean Architecture.
- Nunca adicionar dependências sem justificar.
- Priorizar simplicidade sobre complexidade.
- Em caso de dúvida, perguntar antes de implementar.