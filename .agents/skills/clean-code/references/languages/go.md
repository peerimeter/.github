# Go

Apply alongside [catalog.md](../catalog.md). Go has no exceptions and favors small, explicit
code, so several generic rules translate rather than apply literally.

## Tooling (E1, E2, G24)

- Format with `gofmt` (or `goimports`); vet with `go vet`; lint with `staticcheck` or
  `golangci-lint` if the project uses it.
- Test with `go test ./...`; use `-race` for concurrent code and `-cover` for coverage.
- One command builds (`go build ./...`) and one tests (`go test ./...`).

## Language-specific rules

| Code | Rule | Why |
| --- | --- | --- |
| GO1 | Accept interfaces, return concrete types; define interfaces where they are consumed | Keeps abstractions small and DIP-friendly |
| GO2 | Treat errors as values: check every error, wrap with `fmt.Errorf("...: %w", err)`, and never ignore one silently | The Go form of "exceptions with context" (G4) |
| GO3 | Avoid stutter in names (`user.User` becomes `user.Account` or similar); keep package names short and lowercase | Names read at the call site (N1) |
| GO4 | Pass `context.Context` as the first parameter; do not store it in structs | Standard cancellation and deadlines |
| GO5 | Write table-driven tests with named cases and `t.Run` | Compact, readable, boundary-friendly (T5) |
| GO6 | Make the zero value useful where possible | Fewer constructors and flag arguments (F1) |
| GO7 | Use functional options or a config struct for many optional settings | Satisfies F1 and F3 idiomatically |
| GO8 | Avoid `init()` and package-level mutable state | Hidden dependencies (G22) |
| GO9 | Do not start goroutines without a plan to stop them | Concurrency hygiene |

## Idiom notes

- "Prefer exceptions over error codes" translates to "return an `error` and add context";
  do not use `panic` for expected failures.
- Short names (`i`, `r`, `ctx`) are idiomatic for short scopes (N5).
- Early returns and flat code are the norm; guard clauses fit naturally.
- Small packages with clear purposes beat deep trees; avoid `util` and `common` dumping grounds (G17).
