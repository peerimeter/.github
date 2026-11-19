# Kotlin

Apply alongside [catalog.md](../catalog.md). Many generic rules become language features in
Kotlin; use the feature.

## Tooling (E1, E2, G24)

- Build with the project's Gradle or Maven wrapper; one command to build, one to test.
- Format and lint with `ktlint` or `detekt` (and `spotless` if configured); let them decide style.
- Test with JUnit 5 or Kotest; use `kotlinx-coroutines-test` for suspending code.

## Language-specific rules

| Code | Rule | Why |
| --- | --- | --- |
| KT1 | Prefer `val` over `var` and immutable collections | Fewer hidden side effects (N7) |
| KT2 | Model variants as `sealed` types and use exhaustive `when` | Type-safe G23 |
| KT3 | Use `data class` for value carriers | Removes boilerplate (G12) |
| KT4 | Use named and default arguments instead of boolean flags or telescoping overloads | Addresses F1 and F3 |
| KT5 | Never use `!!` in production code; use safe calls, `requireNotNull`, or redesign | Null-safety is the point of the type system |
| KT6 | Use scope functions (`let`, `apply`, `also`, `run`) sparingly and never nested | Obscured intent (G16) |
| KT7 | Use structured concurrency; avoid `GlobalScope` | Lifecycle-bound, cancellable work |
| KT8 | Prefer extension functions for operations on types you do not own | Wraps boundaries (G6) without wrappers everywhere |

## Idiom notes

- Single-expression functions are fine when the expression is short and clear; extract named
  parts when it grows.
- `object` declarations and top-level functions are idiomatic; avoid G18 (inappropriate static)
  by choosing classes where variation is likely.
- Follow the official Kotlin coding conventions; they supersede generic naming advice.
