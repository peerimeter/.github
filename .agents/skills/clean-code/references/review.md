# Review Workflow

Use this when asked to review code, a diff, a pull request, or "check this for clean-code
issues". The goal is a short, prioritized, actionable list, not an exhaustive audit.

## Steps

1. **Establish scope.** Review only the files or hunks named. For a pull request, read the diff
   and enough surrounding code to judge it. Do not review untouched legacy code unless asked.
2. **Learn the conventions.** Skim neighboring files, the linter and formatter config, and
   contributing docs. Project conventions outrank generic rules.
3. **Run the cheap checks.** If tests, linters, or type checks are available and safe to run,
   run them. Do not report what a tool already reports; mention tool failures once.
4. **Scan in risk order.**
   1. Correctness risks: G3, G4, G26, G31, T1, T5, error handling, null handling.
   2. Structure: G5, G6, G14, G23, G30, G34, F1 to F3, class responsibilities, dependencies.
   3. Readability: N1 to N7, G16, G19, G25, G28, G29, comments (C1 to C5).
   4. Tests: T1 to T9, F.I.R.S.T., one concept per test.
5. **Write findings.** Limit to the issues that matter (roughly ten for a medium diff). Merge
   repeated instances into one finding with a list of locations.
6. **Say what is good.** Name at least one thing done well when it is true.

## Severity

| Level | Meaning |
| --- | --- |
| **Blocker** | Likely bug, data loss, security issue, disabled safety (G4), or a test that cannot catch regressions |
| **Major** | Structural smell that will raise the cost of the next change (G5, G6, G14, G23, G30, F1 to F3) |
| **Minor** | Readability or consistency issue worth fixing in this change (naming, comments, magic values) |
| **Nit** | Taste or tiny polish; mention at most a couple, and mark them optional |

## Output format

````markdown
## Clean-code review: <scope>

**Verdict:** <one sentence: ship, ship after fixes, or needs rework>

### Blocker
- **G4 - tests muted** - `tests/test_billing.py:42`: `@skip` hides a failing case.
  Fix the cause or delete the test with a reason.

### Major
- **G5 - duplication** - `invoice.py:18-31` and `credit_note.py:22-35` compute tax identically.
  Extract `apply_tax(amount, region)`.

### Minor
- **N1 - vague name** - `util.py:7`: `proc(d, f)` -> `render_invoice(invoice, include_tax)`.

### Nit (optional)
- **C3** - `models.py:12` comment repeats the line below.

### Done well
- Clear boundary around the payment gateway; the adapter keeps vendor types out of the domain.
````

Rules for the format:

- One line of *what and where*, one line of *how to fix*. Add a code sample only when it saves
  words.
- Always cite the rule code and a `path:line` location.
- Do not invent line numbers; if you cannot pin one, name the function.
- Omit empty severity sections.
- Phrase findings about the code, not the author.

## What not to flag

- Anything the project's formatter or linter already enforces.
- A rule the surrounding code deliberately breaks for a documented reason.
- Pre-existing issues outside the change, unless they make the change unsafe. Offer them as an
  optional follow-up instead.
- Style preferences that are not backed by a rule or the project's conventions.

## Applying fixes after a review

If the user asks you to apply the fixes, switch to [refactor.md](refactor.md): fix Blockers
first, then Majors, one finding at a time, and re-run the tests between steps.
