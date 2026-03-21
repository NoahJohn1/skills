# .NET Stack Module

These rules are ENFORCED, not recommendations. When this module is loaded, every rule below must be included as a constraint in every delegation prompt. Violations found during review are critical issues requiring re-implementation.

**Build and test commands**: Invoke the `/dotnet-build-test` skill for command discovery. Detected commands must be passed to every implementer and tester delegation.

## Framework Detection

- Projects may target .NET Framework (4.x) OR .NET 6/7/8/9/10. Never assume which version.
- Check the `.csproj` `<TargetFramework>` element BEFORE delegating any work.
- The detected target framework version must appear in every delegation prompt.

## Stack Rules

- **Data Access**: Dapper ONLY. No Entity Framework. No stored procedures. All SQL lives in repository classes as parameterized inline queries.
- **Testing**: MSTest. Use `[TestClass]`, `[TestMethod]`, `[DataRow]`.
- **DI**: Microsoft.Extensions.DependencyInjection for .NET Core+. Unity/Autofac/Ninject possible in Framework projects — check existing registrations.
- **Nullable reference types**: Follow whatever the project has configured. Do not change the setting.
- **async/await**: Use throughout for I/O operations in .NET Core+ projects. In Framework projects, follow existing sync/async patterns.
- **Naming**: PascalCase for public members, _camelCase for private fields, camelCase for parameters and locals. Follow existing project conventions if they differ.
- **File structure**: One class per file. File name matches class name. Namespace matches folder structure.
- **Logging**: `ILogger<T>` for .NET Core+. Check existing logging pattern in Framework projects.

## Architecture — Controller / Service / Repository

Mandatory layering. No exceptions.

### Controller Layer
- Thin. HTTP concerns only: model binding, validation attributes, error handling, response shaping.
- NO business logic. Controllers call services and return results.
- Try/catch at controller level returns appropriate HTTP status codes.
- One public method per endpoint. No private helper methods containing logic.

### Service Layer
- ALL business logic lives here. This is the most important layer.
- Services receive DTOs or primitives, never HttpContext or Request objects.
- Services call repositories for data or api access — never raw SQL or database connections.
- Services are the primary test target. Must have thorough unit tests with mocked repositories.
- Services throw typed exceptions (not HTTP exceptions) — controllers translate these to status codes.

### Repository Layer
- Simple data access. CRUD operations. No business logic. No conditional branching based on data content.
- Returns domain models or DTOs. Never returns DataReader, DataTable, or raw database types to callers.
- One repository per aggregate root / primary entity.
- Dapper only. Parameterized inline SQL. No string concatenation. No stored procedures.
- No Entity Framework. No DbContext. No LINQ-to-entities.

### Layer Rules
- Dependencies flow DOWN only: Controller → Service → Repository
- NEVER skip layers. Controllers do not call repositories directly.
- NEVER reference up. Repositories do not know about services. Services do not know about controllers.
- Interfaces for all services and repositories. Registered in DI container.
- No static classes with business logic. No "Helper" or "Utility" classes that secretly contain domain rules.

## Testing Requirements

- Service layer tests are MANDATORY and must be thorough — this is where business logic lives.
- Controller tests are lightweight (verify status codes, error handling).
- Repository tests are integration tests (only if test database is available).
- If tests fail, delegate back to implementer with failure details. Testers do NOT fix code.

## Review Checklist

Reviewer MUST specifically check for:
- Layer violations (controller calling repository, business logic in wrong layer)
- Entity Framework usage (must not exist)
- Stored procedure usage (must not exist)
- Business logic in controllers or repositories
- Proper DI registration for all new services and repositories
- Raw SQL outside repository classes
- String-concatenated SQL (must use parameterized queries)
