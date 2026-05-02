# Copilot CLI Build Plan — BeSolid.AspireWeather (.NET 10 + Aspire)

This document defines a step-by-step execution plan for building the AspireWeather solution using GitHub Copilot CLI. It includes implementation guidance, CI/CD, and testing conventions to be used when executing each step.

---

# 1. Context

We are building a .NET 10 solution using Aspire and Aspire AppHost.

System contains:

- One microservice: Weather.Api
- Clean Architecture (Domain / Application / Infrastructure)
- CQRS with MediatR
- Vertical Slice architecture inside Application
- SQL Server via Aspire (containerized for local/dev and ephemeral for CI)
- DbUp for database migrations (versioned, idempotent scripts)
- Swagger for API documentation

---

# 2. Repository Structure

The repository uses a `src` folder at the root. All projects MUST be placed under `/src`.

Resulting structure (example):

- /src/BeSolid.AspireWeather.AppHost            (.NET console/host)
- /src/BeSolid.AspireWeather.ServiceDefaults    (shared defaults/config)
- /src/BeSolid.AspireWeather.Weather.Api        (ASP.NET Web API)
- /src/BeSolid.AspireWeather.Weather.Domain     (domain models & value objects)
- /src/BeSolid.AspireWeather.Weather.Application (application services, MediatR handlers)
- /src/BeSolid.AspireWeather.Weather.Infrastructure (EF/SQL, repo implementations)
- /src/BeSolid.AspireWeather.Weather.DbUp       (DbUp console app, migration scripts)

Include a single solution file: BeSolid.AspireWeather.sln at repo root.

---

# 3. Versioning, SDKs and Templates (IMPORTANT)

- Target SDK: .NET 10 (explicitly document CLI setup: dotnet --version requirement)
- Solution file name: BeSolid.AspireWeather.sln
- Project templates:
  - AppHost: dotnet new console
  - Api: dotnet new webapi (minimal Program.cs ok)
  - DbUp: dotnet new console
  - Class libraries: dotnet new classlib for Domain/Application/Infrastructure

---

# 4. Namespace Rules (IMPORTANT)

All namespaces in ALL projects MUST start with:

```
BeSolid.AspireWeather
```

Examples:

- BeSolid.AspireWeather.Weather.Api
- BeSolid.AspireWeather.Weather.Application
- BeSolid.AspireWeather.Weather.Domain

Rules:
- No exceptions allowed
- Folder structure should match namespaces where applicable
- Generated code must respect the root namespace

---

# 5. Architecture Rules

- Weather.Api is the only microservice
- Domain has no external dependencies
- Application contains all business logic (CQRS)
- Infrastructure contains persistence and external integrations
- DbUp is responsible for database schema creation only (no runtime business logic)
- Aspire AppHost orchestrates services and startup order
- API must remain thin (controllers only + MediatR calls)
- Features must be organized using vertical slices
- Add health-check endpoints and readiness/liveness probes
- Add structured logging and OpenTelemetry tracing hooks in AppHost and Weather.Api

---

# 6. Execution Plan (Copilot CLI Steps)

Each step should be executed using the Copilot CLI. Use small iterative prompts, review generated changes, and commit through your normal git workflow.

Example prompt:
```
copilot-cli suggest "Create solution and projects under /src with root namespace BeSolid.AspireWeather targeting .NET 10; include minimal Program.cs and csproj files."
```

## STEP 1 — Solution Setup (with src folder + namespaces)

Goals:
- Create solution file BeSolid.AspireWeather.sln
- Create projects under /src with the templates above and correct root namespaces
- Add minimal Program.cs for WebAPI and AppHost
- Add README note with dotnet SDK version and local dev prerequisites

Suggested checks after generation:- dotnet restore and dotnet build succeed

---

## STEP 2 — Aspire Orchestration (AppHost)

Goals:
- Configure Aspire AppHost to orchestrate SQL Server and Weather.Api
- Add SQL Server container definition and WaitFor rules
- Add service references and wiring sample in AppHost (example registration code snippet)

Add a brief AppHost wiring example in the docs for developers to follow.

---

## STEP 3 — Database Setup

Goals:
- Add SQL Server container for local development and set DB name: weatherdb
- Store connection string via environment variable and document local dev secrets (dotnet user-secrets recommended)
- Ensure CI uses ephemeral DB and injects connection via GitHub Secrets

---

## STEP 4 — DbUp Project (Migrations)

Goals:
- Create DbUp console project to run embedded SQL scripts
- Enforce script naming: V{000}_Description.sql (zero-padded incremental version) or YYYYMMDD_HHMM_{desc}.sql
- Scripts must be idempotent when possible
- DbUp creates WeatherForecast, OutboxMessages, InboxMessages tables and tracks applied scripts

Add a verification step in DbUp to fail on unexpected schema state.

---

## STEP 5 — Database Seeding (Initial Data)

Goals:
- Add seed scripts in DbUp with idempotent inserts (use MERGE or IF NOT EXISTS)
- Version seed scripts like migrations (e.g., 004_SeedWeatherForecast.sql)
- Document how to re-run safely in dev environment

---

## STEP 6 — Wire DbUp into Aspire

Goals:
- Ensure startup order: SQL Server -> DbUp -> Weather.Api
- Implement WaitFor and retry/backoff for DB readiness
- Verify migrations applied before API starts (AppHost should block until migrations succeed)

---

## STEP 7 — Domain Layer

Goals:
- Implement WeatherForecast entity and value objects in BeSolid.AspireWeather.Weather.Domain
- Keep Domain pure (no framework dependencies)

---

## STEP 8 — Application Layer (CQRS)

Goals:
- Implement MediatR commands/queries/handlers and DTOs under BeSolid.AspireWeather.Weather.Application
- Organize by vertical slices (Features/<FeatureName>/...)

---

## STEP 9 — Pipeline Behaviors

Goals:
- Add MediatR pipeline behaviors: Logging, Validation (FluentValidation), Retry (Polly), and Exception/Telemetry behavior

---

## STEP 10 — Infrastructure Layer

Goals:
- Implement persistence (EF Core or Dapper) in BeSolid.AspireWeather.Weather.Infrastructure
- Provide repository interfaces and concrete implementations; register via DI
- Add health checks for DB connectivity

---

## STEP 11 — API Controllers

Goals:
- Create REST controllers in BeSolid.AspireWeather.Weather.Api using MediatR
- Keep controllers minimal; return proper HTTP status codes and problem details for errors
- Enable Swagger and document endpoints

---

## STEP 12 — Swagger + OpenAPI

Goals:
- Add Swagger to Weather.Api and ensure XML comments are included for better docs
- Publish OpenAPI spec as an artifact in CI

---

## STEP 13 — Vertical Slicing

Goals:
- Refactor Application into vertical slices per feature (Feature folders contain request/handler/validators/dtos)

---

## STEP 14 — Weather Feature

Goals:
- Implement CreateWeatherForecast and GetWeatherForecasts features with handlers, DTOs, controller endpoints and tests

---

## STEP 15 — Dependency Injection

Goals:
- Register MediatR, FluentValidation, Infrastructure services, pipeline behaviors, health checks, and OpenTelemetry in AppHost/Api startup

---

## STEP 16 — Run System (Local Developer Flow)

Goals:
- Start Aspire AppHost which brings up SQL Server, runs DbUp, and starts Weather.Api
- Verify Swagger UI, health endpoints, and DB tables/seeding
- Add a dev-compose or script for quick local development (docker-compose or Aspire equivalent)

---

# 7. CI/CD & Tests

Add CI/CD and testing strategy (GitHub Actions + xUnit + Testcontainers/Pact):

- CI with GitHub Actions: build, run unit tests, run integration tests against ephemeral SQL Server (Testcontainers or Docker service), run contract tests, and publish artifacts.
- Unit tests: xUnit projects co-located with Application/Domain (naming: *.Tests) covering handlers and domain logic.
- Integration tests: use Testcontainers for SQL Server in CI or Docker service, run DbUp migrations as part of test setup, verify migrations and optionally rollback behavior.
- Contract tests: consumer/provider contracts (Pact or similar) for Weather.Api; run provider verification in CI.
- Coverage & Quality: collect coverage (coverlet) and publish; optionally fail CI on threshold. Run static analysis (e.g., dotnet-format, SonarCloud optional).
- Secrets & config: use GitHub Secrets for connection strings; document local dev flow using dotnet user-secrets or .env ignored by git.
- Artifacts & releases: publish build artifacts, test reports, and OpenAPI spec; optional CD job to publish container images when merging to main.
- Example workflow (.github/workflows/ci.yml): checkout, setup .NET 10, restore/build, unit tests with coverage, start ephemeral SQL, run DbUp, run integration tests, run contract tests, publish reports.

---

# 7.1 Feature breakdown (branches & PRs)

The plan has been split into feature work items; each should be implemented on its own branch and PR. Todos were created in the session DB (10 pending).

- feature-solution-skeleton — Solution skeleton: create BeSolid.AspireWeather.sln and initial projects under /src, minimal Program.cs, README/dev-setup.md
- feature-apphost-orchestration — AppHost & Aspire orchestration: implement Aspire AppHost wiring, SQL Server container, WaitFor rules
- feature-dbup-migrations — DbUp migrations & seeding: create DbUp console project, versioned idempotent migrations and seeds
- feature-domain-application — Domain & Application (CQRS): domain entities, MediatR handlers, vertical slices
- feature-infrastructure-di — Infrastructure & DI: persistence implementations, repositories, DI registration, health checks
- feature-api-swagger — API controllers & Swagger: controllers, Swagger/OpenAPI and XML docs
- feature-ci-tests — CI/CD & Tests: GitHub Actions, unit/integration/contract tests with ephemeral DB
- feature-dev-dx — Developer DX & tooling: README/dev-setup, Dockerfiles, dev-compose, global.json, editorconfig
- feature-observability-health — Observability & health: OpenTelemetry, structured logging, health endpoints
- feature-branching-pr — Branching & PR policies: branch naming, PR templates, required checks

---

# 8. Policies, Branching & PR Rules

- Branch naming: feature/<ticket>-short-desc, fix/<ticket>-desc, chore/<desc>
- PR title convention: <scope>: <short description> (#<ticket>)
- Required checks: build, unit tests, integration tests (smoke), contract verification, code coverage report
- Code review: at least one reviewer and passing CI before merge

---

# 9. Developer Experience and Tooling

- Provide README/dev-setup.md with dotnet SDK version, how to run Aspire AppHost, running DbUp, running tests, and secrets setup
- Provide Dockerfiles for Api and DbUp projects and a dev-compose for local development
- Add dotnet-format and editorconfig; recommend IDE settings

---

# 10. Execution Workflow (Using Copilot CLI)

For each step:
1. Run Copilot CLI prompt (small, specific instructions)
2. Review generated changes carefully
3. Accept, edit, and commit locally
4. Run dotnet restore/build and tests
5. Push branch and open PR following branch/PR rules

---

# 11. End Result

You will have:
- One Aspire-managed system
- One microservice (Weather.Api)
- Clean Architecture structure
- CQRS with MediatR and pipeline behaviors
- Vertical slice organization
- Automated database migrations via DbUp with versioning and idempotent scripts
- Seeded database for development/testing
- Swagger-enabled REST API with OpenAPI artifact in CI
- CI pipeline with ephemeral DB for integration tests and contract verification
- Clear branch/PR rules and developer onboarding docs

---

# 11.1 Features → Steps mapping (visible linkage)

Implement each feature on its own branch and PR. Below each feature lists the plan steps it implements and its dependencies (session DB todos track status).

- feature-solution-skeleton (Steps: STEP 1)
  - Creates BeSolid.AspireWeather.sln and the initial projects under /src, minimal Program.cs, and README/dev-setup.md.
  - Depends on: none

- feature-dev-dx (Steps: STEP 1, cross-cutting)
  - Adds global.json (SDK pin), .editorconfig, .gitattributes, Dockerfiles, dev-compose, and developer README.
  - Depends on: feature-solution-skeleton

- feature-branching-pr (Steps: Policies & Branching section)
  - Adds branch/PR templates, branch naming and PR conventions, and required checks documentation.
  - Depends on: feature-solution-skeleton

- feature-apphost-orchestration (Steps: STEP 2, STEP 6, STEP 16)
  - Implements Aspire AppHost wiring, SQL Server container definitions, WaitFor rules, and startup ordering.
  - Depends on: feature-solution-skeleton, feature-dev-dx

- feature-dbup-migrations (Steps: STEP 4, STEP 5, STEP 6)
  - Adds DbUp console project, versioned idempotent migration scripts and seed scripts, and integrates DbUp startup.
  - Depends on: feature-apphost-orchestration, feature-solution-skeleton

- feature-infrastructure-di (Steps: STEP 10, STEP 15)
  - Implements persistence (EF Core/Dapper), repository interfaces, DI registration, and DB health checks.
  - Depends on: feature-dbup-migrations, feature-solution-skeleton

- feature-domain-application (Steps: STEP 7, STEP 8, STEP 13)
  - Implements Domain entities (WeatherForecast), Application MediatR commands/queries/handlers, and vertical-slice organization.
  - Depends on: feature-solution-skeleton

- feature-api-swagger (Steps: STEP 11, STEP 12, STEP 14)
  - Creates API controllers that call MediatR, enables Swagger/OpenAPI, and implements the Weather feature endpoints.
  - Depends on: feature-domain-application, feature-infrastructure-di

- feature-observability-health (Cross-cutting: Architecture rules, STEP 9, STEP 15, STEP 16)
  - Adds structured logging, OpenTelemetry hooks, MediatR telemetry/exception behaviors, and readiness/liveness endpoints.
  - Depends on: feature-apphost-orchestration, feature-api-swagger, feature-infrastructure-di

- feature-ci-tests (Steps: CI/CD & Tests — Section 7)
  - Adds GitHub Actions workflows (build, unit tests, integration tests using ephemeral DB/Testcontainers, contract tests), coverage collection, and publishes artifacts.
  - Depends on: feature-dbup-migrations, feature-infrastructure-di, feature-domain-application, feature-api-swagger, feature-apphost-orchestration

---

# How to use this mapping

1. Pick the next READY feature (no unmet dependencies). Use the session DB to query ready todos.
2. Create a branch: feature/<id>-short-desc (e.g., feature-solution-skeleton).
3. Implement the listed plan steps for that feature using Copilot CLI prompts.
4. Commit, push, and open a PR referencing the feature todo id.
5. After merge, update the corresponding todo status in the session DB to 'done'.

