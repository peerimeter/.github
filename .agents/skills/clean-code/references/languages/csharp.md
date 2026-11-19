# C#

Apply alongside [catalog.md](../catalog.md). .NET conventions are strong and tool-enforced;
where they differ from a generic rule, follow .NET.

## Tooling (E1, E2, G24)

- Format and analyze with `dotnet format`, `.editorconfig`, and the project's Roslyn analyzers.
- Test with xUnit, NUnit, or MSTest as the project uses; run `dotnet test`.
- One command builds (`dotnet build`) and one tests (`dotnet test`).

## Language-specific rules

| Code | Rule | Why |
| --- | --- | --- |
| CS1 | Enable nullable reference types and treat warnings as real findings | Compiler-checked "no null" (G26) |
| CS2 | Use `async`/`await` end to end; pass `CancellationToken`; avoid `.Result` and `.Wait()` | Prevents deadlocks and hidden blocking |
| CS3 | Use `record` types for immutable data and value equality | Replaces boilerplate DTOs (G12) |
| CS4 | Use `using` for `IDisposable` and `IAsyncDisposable` | Reliable cleanup |
| CS5 | Use constructor injection and the built-in DI container; bind settings with the options pattern | Construction separated from use |
| CS6 | Use `enum`, sealed hierarchies, or pattern matching for variants | Type-safe G23 |
| CS7 | Throw specific exceptions with context; do not catch `Exception` and swallow it | G4, error handling |
| CS8 | Prefer expression-bodied members only when the expression is short and obvious | Intent (G16) |

## Idiom notes

- The `I` prefix on interfaces and `Async` suffix on async methods are .NET conventions. They
  are accepted exceptions to N6 and N7; follow them for consistency (G11, G24).
- `PascalCase` for public members and `_camelCase` for private fields are standard; follow the
  repository's `.editorconfig`.
- LINQ is excellent for queries; extract a method or name intermediate results when a chain
  stops being readable.
- Keep controllers and minimal-API handlers thin; push logic into services.
