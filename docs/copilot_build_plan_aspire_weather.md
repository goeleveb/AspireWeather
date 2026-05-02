# Copilot CLI Build Plan — BeSolid.AspireWeather (.NET 10 + Aspire)

This document defines a step-by-step execution plan for building the AspireWeather solution using GitHub Copilot CLI.

---

# 1. Context

We are building a .NET 10 solution using .NET Aspire.

System contains:

- One microservice: Weather.Api
- Clean Architecture (Domain / Application / Infrastructure)
- CQRS with MediatR
- Vertical Slice architecture inside Application
- SQL Server via Aspire
- DbUp for database migrations
- Swagger for API documentation

---

# 2. Repository Structure

The repository uses a `src` folder at the root.

All projects MUST be placed under:

```
/src
```

Resulting structure:

- /src/BeSolid.AspireWeather.AppHost
- /src/BeSolid.AspireWeather.ServiceDefaults
- /src/BeSolid.AspireWeather.Weather.Api
- /src/BeSolid.AspireWeather.Weather.Domain
- /src/BeSolid.AspireWeather.Weather.Application
- /src/BeSolid.AspireWeather.Weather.Infrastructure
- /src/BeSolid.AspireWeather.Weather.DbUp

---

# 3. Namespace Rules (IMPORTANT)

All namespaces in ALL projects MUST start with:

```
BeSolid.AspireWeather
```

### Examples:

- BeSolid.AspireWeather.Weather.Api
- BeSolid.AspireWeather.Weather.Application
- BeSolid.AspireWeather.Weather.Domain
- BeSolid.AspireWeather.Weather.Infrastructure
- BeSolid.AspireWeather.Weather.DbUp
- BeSolid.AspireWeather.AppHost
- BeSolid.AspireWeather.ServiceDefaults

### Rules:

- No exceptions allowed
- Folder structure must match namespace structure where applicable
- All generated code must respect this root namespace

---

# 4. Architecture Rules

- Weather.Api is the only microservice
- Domain has no external dependencies
- Application contains all business logic (CQRS)
- Infrastructure contains persistence and external integrations
- DbUp is responsible for database schema creation only
- Aspire AppHost orchestrates all services
- API must remain thin (controllers only + MediatR calls)
- Features must be organized using vertical slices

---

# 5. Execution Plan (Copilot CLI Steps)

Each step should be executed using:

```bash
copilot-cli suggest "<instruction>"
```

---

## STEP 1 — Solution Setup (with src folder + namespaces)

Create solution and projects under `/src` using namespace root `BeSolid.AspireWeather`.

---

## STEP 2 — Aspire Orchestration

Configure AppHost:

- Add SQL Server container
- Add Weather.Api service
- Configure service references
- Configure WaitFor dependencies

---

## STEP 3 — Database Setup

- Add SQL Server in Aspire
- Create database: weatherdb
- Share connection string with Weather.Api

---

## STEP 4 — DbUp Project (Migrations)

Create DbUp console project:

- Runs embedded SQL scripts
- Creates WeatherForecast table
- Creates OutboxMessages and InboxMessages tables

---

## STEP 5 — Database Seeding (Initial Data)

Add seed data using DbUp scripts:

- Insert initial WeatherForecast records
- Ensure idempotent inserts (safe re-runs)
- Version scripts (e.g. 004_SeedWeatherForecast.sql)

---

## STEP 6 — Wire DbUp into Aspire

Ensure startup order:

- SQL Server starts first
- DbUp runs second
- Weather.Api starts last

Use WaitFor dependencies

---

## STEP 7 — Domain Layer

Create:

- WeatherForecast entity

Namespace MUST be:

```
BeSolid.AspireWeather.Weather.Domain
```

---

## STEP 8 — Application Layer (CQRS)

Namespace MUST be:

```
BeSolid.AspireWeather.Weather.Application
```

Implement:

- MediatR commands
- MediatR queries
- Handlers
- DTOs

---

## STEP 9 — Pipeline Behaviors

Add MediatR behaviors:

- Logging
- Validation (FluentValidation)
- Retry (Polly)

---

## STEP 10 — Infrastructure Layer

Namespace MUST be:

```
BeSolid.AspireWeather.Weather.Infrastructure
```

Implement:

- Persistence logic for WeatherForecast
- External integrations (if needed)
- Repository pattern or direct database access implementation

---

## STEP 11 — API Controllers

Namespace MUST be:

```
BeSolid.AspireWeather.Weather.Api
```

Create REST controllers using MediatR.

---

## STEP 12 — Swagger

Enable Swagger in Weather.Api.

---

## STEP 13 — Vertical Slicing

Refactor Application layer into vertical slices.

---

## STEP 14 — Weather Feature

Implement:

- CreateWeatherForecast
- GetWeatherForecasts

---

## STEP 15 — Dependency Injection

Register:

- MediatR
- FluentValidation
- Infrastructure services
- Pipeline behaviors

---

## STEP 16 — Run System

Run full solution in Visual Studio 2026:

- Aspire AppHost
- SQL Server container
- DbUp migrations + seeding
- Weather.Api
- Verify Swagger UI

---

# 5.1 CI/CD & Tests

Add CI/CD and testing strategy (GitHub Actions + xUnit/Pact):

- CI with GitHub Actions: build, run unit tests, run integration tests (ephemeral SQL Server + DbUp migrations), run contract tests, and publish artifacts.
- Unit tests: create xUnit projects co-located with Application/Domain projects (naming: *.Tests) to cover business logic and handlers.
- Integration tests: run against a throwaway SQL Server (Docker action/service). Integration test setup MUST run DbUp migrations to bring schema to expected version and seed required test data. Tests should verify migrations applied and include a migration-rollback verification step where feasible.
- Contract tests: add consumer/provider contracts (e.g., Pact or HTTP contract tests) for Weather.Api to validate API surface with consumers. Run provider verification in CI.
- Coverage & Quality: collect code coverage (e.g., coverlet) and fail CI if below threshold (configurable). Run static analysis/formatters as needed.
- Secrets & config: use GitHub Secrets for connection strings; CI should inject them as env vars. Document local dev secrets workflow (dotnet user-secrets or env files excluded from repo).
- Artifacts & releases: publish build artifacts and test reports (JUnit/coverage) from CI; include an optional CD job to publish container images or releases when on main branch.
- Example workflow: add .github/workflows/ci.yml template that:
  1. Checks out code and sets up .NET 10
  2. Restores and builds solution
  3. Runs unit tests with coverage
  4. Starts SQL Server service (or Docker container), waits for readiness
  5. Runs DbUp migrations (console project) against the ephemeral DB
  6. Runs integration tests against the migrated DB
  7. Runs contract/provider verification tests
  8. Publishes test results and coverage reports

# 6. Execution Workflow

For each step:

1. Run Copilot CLI prompt
2. Review generated changes
3. Accept or adjust
4. Proceed to next step

---

For each step:

1. Run Copilot CLI prompt
2. Review generated changes
3. Accept or adjust
4. Proceed to next step

---

# 7. End Result

You will have:

- One Aspire-managed system
- One microservice (Weather.Api)
- Clean Architecture structure
- CQRS with MediatR
- Vertical slice organization
- Automated database migrations via DbUp
- Seeded database for development/testing
- Swagger-enabled REST API
- All projects under `/src`
- All namespaces prefixed with `BeSolid.AspireWeather`

