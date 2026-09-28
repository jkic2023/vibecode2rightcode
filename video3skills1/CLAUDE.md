# CLAUDE.md

Project Management API: projects that contain tasks. ASP.NET Core Web API (controllers) on .NET 10, EF Core with SQLite. Layers: Domain, Application, Infrastructure, Api.

## Commands

- Run: `dotnet run --project src/ProjectManagement.Api`
- Test: `dotnet test`
- Add a migration: `dotnet ef migrations add <Name> --project src/ProjectManagement.Infrastructure --startup-project src/ProjectManagement.Api --output-dir Persistence/Migrations`

## Project rules

- Dependencies point inward: Api and Infrastructure depend on Application, Application depends on Domain, Domain depends on nothing. Application never references EF Core. Api references Infrastructure only in `Program.cs`.
- Controllers only translate HTTP: no business logic, no data access.
- Business logic lives in Application services and the Domain. `Project` is the aggregate root, and tasks change only through it.
- Services return a `Result` for expected failures (validation, not found, conflict). The Api turns them into ProblemDetails in one place. Do not throw for expected failures.
- Controllers return DTOs, never entities. Map by hand with `ToDto()` extensions, with no mapper library.
- Application code gets the time from `IDateTime`, never `DateTime.Now` or `DateTime.UtcNow`.
- Never edit an applied migration by hand.

## Skills (in .claude/skills): use by name before writing code

Do not rely on memory for .NET patterns. For each kind of work below, use the named Skill first, then follow it.

- C# style and safety: csharp-coding-standards, csharp-nullable-reference-types
- Concurrency and background work: csharp-concurrency-patterns
- Data access: efcore-patterns, database-performance
- Dependency injection and configuration: microsoft-extensions-dependency-injection, microsoft-extensions-configuration
- Tests: snapshot-testing

If a Skill and a project rule above disagree, the project rule wins. Say so when it happens.
