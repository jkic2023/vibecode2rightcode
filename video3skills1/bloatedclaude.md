# CLAUDE.md

This file tells Claude how we build software here. Read all of it before starting any task and follow every rule.

## Project overview

This is a REST API built with ASP.NET Core. It is a layered solution with an Api project, an Application project, a Domain project and an Infrastructure project, plus test projects. We value clean, maintainable, secure and well-tested code.

## Tech stack

- .NET 10 and C# 14
- ASP.NET Core Web API
- Entity Framework Core with a relational database
- MediatR for commands and queries
- FluentValidation for input validation
- Serilog for logging
- xUnit, Moq and FluentAssertions for tests
- WebApplicationFactory for integration tests
- Swagger/OpenAPI for documentation

## Solution structure

```
src/
  Api/              controllers, middleware, Program.cs
  Application/      commands, queries, handlers, validators, DTOs
  Domain/           entities, value objects, enums
  Infrastructure/   DbContext, migrations, external services
tests/
  UnitTests/
  IntegrationTests/
  ApiTests/
docs/
scripts/
```

## Commands

- Build: `dotnet build`
- Run: `dotnet run --project src/Api`
- Test: `dotnet test`
- Format: `dotnet format`
- Add a migration: `dotnet ef migrations add <Name> -p src/Infrastructure -s src/Api`
- Apply migrations: `dotnet ef database update -p src/Infrastructure -s src/Api`

## General principles

- Write clean, readable code.
- Follow SOLID and keep methods and classes small.
- Do not repeat yourself, but do not over-abstract either.
- Prefer composition over inheritance.
- Use meaningful names.
- Think about edge cases and failure modes.
- Do not add features that were not asked for.
- Prefer simple solutions.
- Keep changes small and focused.
- If a requirement is unclear, ask before guessing.

## C# style

- 4 spaces for indentation, never tabs.
- Opening braces on a new line.
- File-scoped namespaces.
- One type per file.
- PascalCase for types, methods and properties. camelCase for parameters and locals.
- Private fields start with an underscore.
- Interfaces start with `I`.
- Async methods end with `Async`.
- Maximum line length 120.
- Always use braces for `if`, `for` and `while`.
- Sort `using` directives: System first, then third party, then ours.
- Use `var` when the type is obvious from the right-hand side.
- Use `nameof` instead of string literals for names.
- Use `is null` and `is not null`.
- Use string interpolation instead of `string.Format`.
- Do not leave commented-out code.
- Add XML documentation to public members.

## Language features

- Nullable reference types are enabled. Do not disable them and do not suppress warnings with `!`.
- Treat warnings as errors.
- Use records for DTOs and value objects.
- Use pattern matching and switch expressions.
- Use primary constructors and collection expressions in new code.
- Do not return null for collections. Return an empty collection.
- Use `TimeProvider` instead of `DateTime.Now`.

## Architecture

- Follow Clean Architecture. Domain has no dependencies. Application depends on Domain. Infrastructure depends on Application and Domain. Api depends on Application and Infrastructure.
- Controllers are thin. They translate HTTP to a command or query and back.
- Business logic lives in Application handlers and in the Domain.
- Every command and query goes through MediatR.
- Map entities to DTOs with explicit mapping methods.
- Use the Result pattern for expected failures. Use exceptions for unexpected ones.
- Keep the Domain free of framework types.
- Follow CQRS: commands change state, queries do not.
- Aggregates are modified only through their own methods.
- Do not reference Infrastructure from Application.

## API design

- Follow REST conventions and use plural nouns for resources.
- Use the right verb: GET reads, POST creates, PUT replaces, PATCH updates, DELETE removes.
- Return 200, 201, 204, 400, 401, 403, 404, 409 and 500 appropriately.
- Return ProblemDetails for errors.
- Never return entities from controllers. Return DTOs.
- Version the API in the URL.
- Every list endpoint supports paging with sensible defaults and a maximum page size.
- Support filtering and sorting on list endpoints.
- Use camelCase in JSON and kebab-case in URLs.
- Document every endpoint with XML comments and OpenAPI attributes.
- Return `Location` headers on 201 responses.
- Use `[ApiController]` and `ActionResult<T>`.
- Accept a `CancellationToken` in every action.

## Data access

- Use EF Core for all data access.
- Use code-first migrations and never edit an applied migration by hand.
- Use `AsNoTracking()` for read-only queries.
- Use `Select` projections instead of loading whole entities when only some fields are needed.
- Avoid N+1 queries. Use `Include` deliberately.
- Use `AsSplitQuery()` when including several collections.
- Filter in the database, not in memory.
- Use `ExecuteUpdate` and `ExecuteDelete` for bulk changes.
- Configure entities with `IEntityTypeConfiguration<T>`.
- Use transactions for multi-step writes.
- Call `SaveChangesAsync` once per unit of work.
- Register `DbContext` as scoped and never share it between threads.
- Enable retry on failure for transient database errors.
- If you write raw SQL, always parameterise it.
- Index columns that are used in filters and joins.
- Keep seed data out of migrations.

## Dependency injection

- Use constructor injection.
- Do not use the service locator pattern.
- Register services in one extension method per layer.
- Scoped for anything that uses the database. Singleton for stateless services. Transient for lightweight services.
- Never inject a scoped service into a singleton.
- Use `IHttpClientFactory` and typed clients for outgoing HTTP.
- Depend on interfaces for anything that talks to the outside world.

## Configuration

- Use the options pattern for all settings, one class per section.
- Validate options on startup.
- Do not read `IConfiguration` in business code.
- Never hardcode connection strings, keys or URLs.
- Never commit secrets. Use user secrets locally and environment variables in production.
- Use `appsettings.{Environment}.json` for environment differences.
- Use `IOptionsSnapshot<T>` for per-request settings and `IOptionsMonitor<T>` in singletons.

## Async and concurrency

- Use async and await for all I/O.
- Never block on tasks with `.Result` or `.Wait()`.
- Never use `async void` except for event handlers.
- Pass a `CancellationToken` through every async call.
- Use `Task.WhenAll` for independent work.
- Do not use `Task.Run` in request handlers.
- Use `Channel<T>` for producer and consumer work and `BackgroundService` for long-running work.
- Use `SemaphoreSlim` instead of `lock` around async code.
- Do not fire and forget.

## Validation and error handling

- Validate all input with FluentValidation.
- Return 400 with the validation errors.
- Use a single global exception handler.
- Log each exception once, where it is handled.
- Never swallow exceptions and never catch `Exception` except at the top level.
- Never expose stack traces to clients.
- Use domain exceptions or Results for business rule failures, and map them to status codes in one place.

## Logging

- Use Serilog with structured logging and message templates.
- Choose log levels carefully: Debug for detail, Information for state changes, Warning for recoverable problems, Error for failures.
- Never log secrets, tokens or personal data.
- Include a correlation ID in every log entry.
- Use `ILogger<T>`. Do not use `Console.WriteLine`.

## Testing

- Write tests for new behaviour and for bug fixes.
- Follow Arrange, Act, Assert.
- Name tests `Method_Scenario_ExpectedResult`.
- Keep tests independent and repeatable.
- Unit tests for domain and handlers. Integration tests for data access. API tests with `WebApplicationFactory`.
- Use `WebApplicationFactory` with a real database, not mocks, for integration tests.
- Use Moq only for boundaries you own.
- Use FluentAssertions for assertions.
- Do not test private methods or the framework.
- Use snapshot tests for large responses.
- Tests must not depend on execution order.
- Aim for meaningful coverage of business logic, not a number.

## Security

- HTTPS everywhere.
- JWT bearer authentication and policy-based authorization.
- Hash passwords with the framework hasher.
- Validate all input and use parameterised queries.
- Restrict CORS to known origins.
- Rate limit public endpoints.
- Set security headers.
- Follow the OWASP Top 10.
- Keep dependencies up to date and check for vulnerable packages before a release.

## Performance

- Paginate.
- Cache reference data that rarely changes.
- Avoid large allocations in hot paths.
- Use `StringBuilder` in loops.
- Use compiled queries for hot EF Core paths.
- Measure before optimising, and use BenchmarkDotNet for micro-benchmarks.

## Serialization

- Use System.Text.Json with shared, preconfigured options.
- Use camelCase, ignore nulls when writing, and never serialise entities directly.

## Git and pull requests

- Branch names: `feature/<ticket>-description` and `bugfix/<ticket>-description`.
- Commit messages in the imperative mood, first line at most 72 characters.
- Never commit directly to `main`.
- Keep pull requests small and link the ticket.
- Run the tests before pushing.
- Do not commit generated files.

## Documentation

- Update the README when the way to run the project changes.
- Record significant decisions as short ADRs in `docs/adr`.
- Comment the why, not the what.
- Update this file when a rule changes.

## Final reminders

Follow all of the rules above. Write secure, maintainable, well-tested code. If you are unsure about something, ask.
