# Pharos

> AI-powered incident triage, classification, and escalation platform built with .NET, event-driven architecture, and Generative AI.

## Overview

Pharos is a portfolio project focused on applying modern Software Engineering and AI Engineering practices to a real-world operational problem: incident management.

The system receives incident reports, analyzes and enriches them using Large Language Models (LLMs), detects duplicates, suggests ownership, orchestrates escalation workflows, and assists operators through intelligent recommendations.

The project is designed to explore concepts such as:

- Backend Engineering with .NET
- Event-Driven Architecture
- Generative AI Integration
- Retrieval-Augmented Generation (RAG)
- Vector Databases
- LLM Evaluation
- Observability
- AI Safety
- MLOps / LLMOps Practices

---

## Architecture

Pharos follows Clean Architecture principles and separates concerns between domain, application, infrastructure, and API layers.

```text
┌─────────────────────┐
│ External Channels   │
│ Telegram / APIs     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│      Pharos API     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│    Application      │
│ Use Cases & Flows   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│       Domain        │
│ Business Rules      │
└──────────┬──────────┘
           │
           ▼
┌────────────────────────────────┐
│ Infrastructure                  │
│ PostgreSQL                      │
│ RabbitMQ                        │
│ Qdrant                          │
│ Ollama / LLM Providers          │
│ GitHub Issues / External APIs   │
└────────────────────────────────┘
```

---

## Core Features

### Incident Management

- Incident registration
- Incident classification
- Priority assignment
- Escalation workflow
- State transitions
- Audit trail

### AI-Assisted Triage

- Automatic categorization
- Information extraction
- Duplicate incident detection
- Team recommendation
- Guided clarification questions
- Structured LLM outputs

### Knowledge Retrieval (RAG)

- Document ingestion
- Chunking and indexing
- Embedding generation
- Semantic search
- Context retrieval
- Source citation

### Event-Driven Processing

- RabbitMQ integration
- Asynchronous workflows
- Retry policies
- Dead Letter Queue (DLQ)
- Delayed processing

### Integrations

- Telegram Bot
- GitHub Issues
- Jira (planned)
- External ticketing providers

### Observability

- Structured logging
- Distributed tracing
- Metrics collection
- Dashboards
- Failure investigation

---

## Technology Stack

### Backend

- .NET 10
- ASP.NET Core
- Entity Framework Core
- C#

### Data

- PostgreSQL
- Qdrant

### Messaging

- RabbitMQ

### Artificial Intelligence

- Microsoft.Extensions.AI
- Ollama
- Local LLMs
- Embeddings
- RAG

### Observability

- OpenTelemetry
- Jaeger
- Prometheus
- Grafana
- Serilog

### Testing

- xUnit
- FluentAssertions
- Testcontainers
- WireMock.Net

### DevOps

- Docker
- Docker Compose
- GitHub Actions

---

## Project Structure

```text
src/
├── Pharos.Api
├── Pharos.Application
├── Pharos.Contracts
├── Pharos.Domain
└── Pharos.Infrastructure

tests/
├── Pharos.Application.Tests
├── Pharos.Domain.Tests
└── Pharos.IntegrationTests

docs/
├── adr
├── architecture
├── ai
└── diagrams
```

---

## Development Roadmap

### Phase 0
- Project foundation
- Architecture decisions
- Development environment

### Phase 1
- Domain modeling
- Persistence layer
- Entity rules

### Phase 2
- API foundation
- Configuration
- Validation

### Phase 3
- Messaging infrastructure
- Event handling

### Phase 4
- LLM integration
- Structured outputs

### Phase 5
- RAG pipeline
- Vector search

### Phase 6
- Intelligent triage agent

### Phase 7
- External integrations

### Phase 8
- End-to-end escalation workflow

### Phase 9
- Security hardening

### Phase 10
- Observability

### Phase 11
- AI evaluation and benchmarking

### Phase 12
- CI/CD pipeline

### Phase 13
- Documentation and final delivery

---

## Running Locally

### Prerequisites

- .NET SDK 10
- Docker Desktop
- Git

### Clone Repository

```bash
git clone https://github.com/<your-user>/pharos.git
cd pharos
```

### Start Infrastructure

```bash
docker compose up -d
```

### Restore Packages

```bash
dotnet restore
```

### Build Solution

```bash
dotnet build
```

### Run Tests

```bash
dotnet test
```

---

## Engineering Principles

This project emphasizes:

- Clean Architecture
- SOLID Principles
- Domain-Driven Design
- Testability
- Observability
- Security by Design
- AI Reliability
- Reproducibility

---

## AI Engineering Goals

Pharos is intended to demonstrate practical AI Engineering skills, including:

- Prompt Engineering
- Retrieval-Augmented Generation (RAG)
- Embedding Pipelines
- Vector Databases
- Function Calling
- Agent Design
- LLM Evaluation
- AI Safety and Guardrails
- LLMOps

---

## Disclaimer

This project is an educational and portfolio-oriented application created to explore software engineering and artificial intelligence concepts. It is not intended for production use without additional security, operational, and compliance reviews.
