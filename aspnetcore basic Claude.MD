# CLAUDE.md

Standing engineering guardrails for any ASP.NET Core work in this repository. This file does not describe any specific app or feature — it applies automatically to whatever gets built here, however it's asked for.

## Architecture

- Use a layered architecture: a Domain layer, an Application layer, an Infrastructure layer, and a Web (presentation) layer, each its own project in the solution, plus a matching test project per layer.
- **Domain layer:** entities and value objects only, no dependency on any other layer. Entities enforce their own invariants — use private setters and behavior methods (e.g. `ChangeStatus()`, not a public setter) rather than letting external code mutate state directly. Throw a domain-specific exception when an invariant is violated.
- **Application layer:** use-case/service logic only, depends on Domain. Define any interfaces it needs from Infrastructure (repositories, unit of work) here — Infrastructure implements them, never the reverse.
- **Infrastructure layer:** EF Core, persistence, and any external integrations. Implements the interfaces defined in Application.
- **Web layer:** controllers only translate a request into a call to the Application layer and back. No business logic, no direct database access, no direct reference to Infrastructure from a controller.

## API / data design

- Controllers must never bind directly to a Domain entity or an EF Core entity. Use a dedicated request/form model for input and a dedicated response/view model for output, and map between them explicitly.
- Keep entities out of anything that crosses a layer boundary — Domain entities stay inside Domain and Application; Web never sees one directly.

## Configuration

- All configuration values (connection strings, external settings) must be bound to a strongly-typed options class via the Options pattern, and validated at startup (`ValidateDataAnnotations().ValidateOnStart()`), so a missing or invalid value fails immediately and loudly rather than silently falling back to a default.
- Group all of a layer's service registration behind a single extension method (e.g. `AddInfrastructure(...)`), called once from `Program.cs` — no ad-hoc registration calls scattered through startup.

## Error handling and observability

- Every Application-layer method that can fail returns a typed result, not a thrown exception, for expected failure cases (validation, not-found, conflict). Only genuinely unexpected exceptions should propagate.
- Any unhandled exception is caught in exactly one place and turned into a consistent error response — never let a raw, unhandled exception reach the caller as a bare 500.
- Inject `ILogger` into every Application-layer service and log on entry, on success, and on failure for anything that mutates state. No empty catch blocks, ever.

## Explicitly out of scope for this repo, for now

Do not add any of the following unless separately and explicitly asked for: authentication or authorization, API versioning, rate limiting, CORS configuration, Swagger/OpenAPI, a CQRS/mediator pipeline, domain events, audit logging or interceptors. If a request seems to imply one of these by pattern (e.g. a well-known template name), build the narrowest version of what was literally asked for instead of the broader pattern it resembles.
