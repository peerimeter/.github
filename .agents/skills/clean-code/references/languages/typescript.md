# TypeScript and JavaScript

Apply alongside [catalog.md](../catalog.md). For plain JavaScript, apply the same rules and
use JSDoc types where the project already does.

## Tooling (E1, E2, G24)

- Format with `prettier` or `biome`; lint with `eslint` or `biome`; type-check with
  `tsc --noEmit`.
- Test with the project's runner (`vitest`, `jest`, `node --test`); measure coverage with its
  built-in reporter.
- Prefer one command to build (`npm run build`) and one to test (`npm test`).
- Respect the repository's quote, semicolon, and import-order style; the formatter decides.

## Language-specific rules

| Code | Rule | Why |
| --- | --- | --- |
| TS1 | Keep imports explicit and stable; avoid `import *` namespaces and deep barrel chains | Visible dependencies (G22) and better tree-shaking |
| TS2 | Use literal unions, `as const` objects, or enums for closed sets | Same intent as J3 and G25 |
| TS3 | Do not use `any` at boundaries; take `unknown` and narrow | Keeps type safety where untrusted data enters |
| TS4 | Model variants as discriminated unions with exhaustive `switch` | Type-safe form of G23 |
| TS5 | Prefer `readonly`, `const`, and immutable updates | Fewer hidden side effects (N7, G26) |
| TS6 | Avoid the non-null assertion `!` and unchecked casts `as` | They silence the compiler (G4) |
| TS7 | Never leave a promise floating; `await` it or handle its rejection | Unhandled rejections hide failures |
| TS8 | Use an options object (with a named interface) instead of long positional lists or boolean flags | Satisfies F1 and F3 |
| TS9 | Validate external input at the edge (schema parsing), then trust the types inside | Boundaries (G3) |

## Idiom notes

- Callbacks and array helpers (`map`, `filter`, `reduce`) are fine; stop chaining when the
  pipeline needs a comment, and name the steps.
- `async` functions are the preferred error-handling shape; do not mix callbacks, promises,
  and `async` in one module (G11).
- Use `interface` or `type` as the project does; be consistent (G11).
- In React or similar frameworks, keep components small, extract hooks for logic, and keep
  rendering free of side effects.

## Before and after

```typescript
// before
function process(data: any, verbose: boolean) { /* ... */ }
const total = order.customer.address.country.taxRate * order.amount;
```

```typescript
// after
interface ProcessOptions { verbose: boolean }
function processOrders(orders: Order[], options: ProcessOptions): ProcessResult { /* ... */ }
const total = order.amountWithTax();
```

Fixes: TS3, TS8, G36.
