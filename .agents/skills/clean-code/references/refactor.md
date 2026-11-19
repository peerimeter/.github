# Refactoring Workflow

Use this to clean existing code without changing what it does. Refactoring means changing
structure while preserving behavior; if behavior must change, do that as a separate step.

## Preconditions

1. **Know the safety net.** Find the tests that cover the code and run them. Record the baseline.
2. **No tests?** Write characterization tests for the paths you will touch (see
   [patterns.md](patterns.md)), or tell the user plainly that the refactor is unverified and
   keep it minimal.
3. **Clean working tree.** Refactor on top of committed or stashed work so each step is easy to
   review and revert.
4. **Agree on scope.** Name the file, function, or smell you are targeting. Do not widen it
   mid-flight; note extra findings for later.

## Loop

Repeat in small steps, running the tests after each:

1. Pick **one** smell from [catalog.md](catalog.md). Prefer, in order: dead code (F4, G9,
   C5), duplication (G5), unclear names (N1 to N7), long or multi-level functions (G30, G34),
   flag and selector arguments (F3, G15), misplaced responsibility (G14, G17), type switches
   (G23).
2. Apply the **smallest** transformation that removes it (see the patterns file).
3. Run the tests and the linter. If anything goes red, undo the step rather than debugging a
   large change.
4. Commit or checkpoint when a step is green and self-contained.

## Safe orderings

- Delete before you restructure: unused code is the cheapest win.
- Rename before you extract: good names make the structure visible.
- Extract before you move: shrink functions, then relocate them.
- Remove duplication before introducing abstractions, and only abstract after the third
  occurrence unless the shared concept is obvious.
- Change one dimension at a time: do not rename, move, and reformat in the same step.

## Public APIs and callers

- Before changing a public function, class, route, or schema, search for callers, including
  tests, scripts, docs, and other packages.
- Prefer deprecating and delegating over breaking callers when the code is a published API.
- Update every caller you can see; list those you cannot (external consumers) in the summary.

## Stop conditions

Stop and report when:

- Tests fail and the cause is not obvious after one step back.
- The refactor would need a behavior change to proceed.
- The scope has grown beyond what was agreed.
- You are fixing style that the formatter would handle anyway.

## Report

Finish with a compact summary:

```text
Refactored: src/billing/invoice.py
- Extracted apply_tax() from render() (G30, G5)
- Replaced tax_rate flag with two functions (F3)
- Removed unused import and dead branch (G9)
Verified: pytest tests/billing (42 passed), ruff clean.
Not done: parse() is still 120 lines; separate change recommended.
```

If you could not run the tests, say so first.
