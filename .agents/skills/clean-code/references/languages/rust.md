# Rust

Apply alongside [catalog.md](../catalog.md). The type system already enforces several
heuristics; lean on it.

## Tooling (E1, E2, G24)

- Format with `cargo fmt`; lint with `cargo clippy --all-targets -- -D warnings` if the project
  gates on it.
- Test with `cargo test` (unit tests in `#[cfg(test)]` modules, integration tests in `tests/`,
  and doc tests).
- One command builds (`cargo build`) and one tests (`cargo test`).

## Language-specific rules

| Code | Rule | Why |
| --- | --- | --- |
| RS1 | Return `Result` and propagate with `?`; do not `unwrap` or `expect` in library or production paths without a stated invariant | Error context instead of panics (G4) |
| RS2 | Use newtypes for distinct domain values (`UserId`, `Meters`) | Prevents argument mix-ups and primitive obsession (N4, G26) |
| RS3 | Model variants as `enum` and use exhaustive `match` | Type-safe G23 |
| RS4 | Respect `clippy`; justify any `#[allow(...)]` in a short comment | Overridden safeties (G4) |
| RS5 | Borrow before you clone; do not `clone()` just to appease the borrow checker | Express ownership intent (G16) |
| RS6 | Use a builder or a config struct for many optional parameters | Satisfies F1 and F3 |
| RS7 | Keep `unsafe` blocks tiny and document the invariant they rely on | Isolated risk |
| RS8 | Implement standard traits (`From`, `Display`, `Default`, `FromStr`) rather than ad hoc methods | Predictable APIs (G24) |

## Idiom notes

- `impl` blocks group behavior with data; avoid free functions that take a struct as the first
  parameter (G14).
- Use `Option` instead of sentinel values; it is the language's form of "never return null".
- Module privacy is the main tool for G8; expose the minimum with `pub(crate)` and `pub`.
- Document public items with `///` and examples; doc tests double as T1 coverage.
