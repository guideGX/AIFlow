# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AiFlow is an open-source AI Agent Platform for creating, managing, and deploying AI agents, chatbots, and visual workflows. It is a polyglot codebase (Python 3.11+, Java 21, TypeScript) licensed under Apache 2.0.

## Repository Structure

```
PaiFlow/
├── console/                # Web Console (full-stack)
│   ├── frontend/           # React 18 + TypeScript + Vite (has its own CLAUDE.md)
│   └── backend/            # Java 21 Spring Boot (Maven, 3 modules: commons, hub, toolkit)
├── core/                   # Python microservices (uv package manager)
│   ├── common/             # Shared utilities library (xingchen_utils)
│   ├── agent/              # AI Agent service (FastAPI, has its own CLAUDE.md)
│   ├── workflow/            # Python workflow engine
│   └── plugin/             # Plugin services (aitools, link)
├── core-workflow-java/     # Java workflow engine (Spring Boot 3.5.4)
├── docker/PaiFlow/         # Docker Compose + Dockerfiles for full stack
├── scripts/                # Database init scripts (MySQL, Postgres)
├── makefiles/              # Modular Makefile components
└── Makefile                # Top-level CI/CD orchestrator
```

## Build & Development Commands

### Top-Level (auto-detects active projects via git diff)

```bash
make setup       # One-time: install tools, hooks, branch strategy
make format      # Format code across all detected projects
make lint        # Lint/quality check
make test        # Run tests
make build       # Build all projects
make ci          # Full pipeline: format + check + test + build
make fix         # Auto-fix code issues
make clean       # Clean build artifacts
```

### Frontend (console/frontend/)

```bash
npm run dev          # Vite dev server on port 3000
npm run build        # Production build
npm run quality      # All checks: format:check + lint + type-check
npm run lint:fix     # Auto-fix ESLint errors
npm run type-check   # TypeScript type checking (tsc --noEmit)
```

### Console Backend (console/backend/)

```bash
mvn clean install              # Build all modules
mvn spring-boot:run -pl hub    # Run the hub API server
mvn test                       # Run tests (JUnit 5 + Mockito, H2 in-memory DB)
```

Code quality: Spotless (formatting), Checkstyle, SpotBugs, PMD.

### Python Core Modules (core/*)

Each module uses **uv** as package manager with `pyproject.toml` + `uv.lock`.

```bash
# Agent service (core/agent/)
cd core/agent
python main.py                          # Run main entry point
uvicorn api.app:app --host 0.0.0.0 --port 8000  # FastAPI server
pytest                                  # Run tests (70% coverage threshold)
python scripts/quality_check.py         # All quality checks (Black, isort, Flake8, MyPy, Pylint)

# Common module (core/common/)
cd core/common
pytest                                  # Run tests (90% coverage threshold)

# Plugin modules (core/plugin/aitools/, core/plugin/link/)
cd core/plugin/aitools  # or link
pytest                  # Run tests
```

### Java Workflow Engine (core-workflow-java/)

```bash
cd core-workflow-java
mvn clean package -DskipTests    # Build
mvn test                         # Run tests
```

### Docker (docker/PaiFlow/)

```bash
cd docker/PaiFlow
docker compose up        # Full stack: MySQL 8.4, Redis 7, MinIO, frontend, hub, workflow
```

Services: mysql (3307), redis, minio (9000/9001), console-frontend (3000->1881), console-hub (8081->8080), core-workflow-java (7880).

## Architecture

### Service Communication

The platform runs as a set of services behind a web console:

- **Console Frontend** (React/Vite) talks to **Console Hub** (Java Spring Boot) via REST + SSE
- **Console Hub** orchestrates calls to **Core Agent** (Python/FastAPI) and **Core Workflow** (Java or Python)
- **Core Agent** uses a plugin system (Knowledge, Link, MCP, Workflow plugins) and a workflow engine with node-based execution (Chat, CoT, CoT Process agents)
- All services share **MySQL** (persistence), **Redis** (caching/sessions), **MinIO** (object storage)

### Key Patterns

- **Frontend**: Casdoor SSO with PKCE, multi-space (personal/enterprise) architecture, Zustand + Recoil state, i18n (zh/en), path alias `@/*` -> `src/*`
- **Console Backend**: Spring Security + OAuth2, MyBatis Plus ORM, Lombok + MapStruct, Maven multi-module (commons/hub/toolkit)
- **Python Core**: DDD architecture (API -> Service -> Engine -> Domain -> Repository -> Cache -> Infrastructure), FastAPI with async/await, SQLAlchemy 2.0, Pydantic models, OpenTelemetry tracing
- **Workflow Engine (Java)**: DSL-based node graph execution, SSE callback system, Spring AI integration

### Sub-Module Documentation

Detailed guidance exists in:
- `console/frontend/CLAUDE.md` — React frontend architecture, commands, patterns
- `core/agent/CLAUDE.md` — Python agent service DDD architecture, quality commands, plugin system
