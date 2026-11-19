---
name: clean-code
description: Applies Robert C. Martin's Clean Code principles when writing, reviewing, or refactoring code in any language - intention-revealing names, small single-purpose functions, honest comments, error handling, SRP/OCP/DIP, tests (TDD, F.I.R.S.T.), and the named smell catalog (C, E, F, G1-G36, N, T). Use whenever code is created, edited, reviewed, or refactored; when asked for a clean-code review, cleanup, or "boy scout" pass; or when the user mentions code smells, readability, maintainability, technical debt, or a rule code such as G5 or F3.
license: MIT
compatibility: Works with any agent that supports the Agent Skills standard (Claude Code, GitHub Copilot, OpenCode, Codex, Gemini CLI, Cursor, Windsurf, Amp). Needs only file read access.
metadata:
    author: patbaumgartner
    version: "1.0.0"
    repository: https://github.com/patbaumgartner/clean-code-skill
---

# Clean Code

Make code that is easy to read, change, and test. Finish the requested task first, then leave
the code you touched a little cleaner than you found it (the Boy Scout Rule).

## Operating rules

1. **Scope discipline.** Clean up only what you touch or what blocks the task. Never start an
   unrequested rewrite, and never mix behavior changes with pure refactoring in one step.
2. **Project conventions win.** Existing style, linters, and formatters beat these rules (G11,
   G24). Match the surrounding code; do not reformat unrelated lines.
3. **Rules are heuristics.** "Max three arguments" and "small functions" guide judgment; they
   are not laws. Explain a deliberate exception in one short line instead of contorting code.
4. **Prove behavior is preserved.** Refactor only under passing tests. If none exist, add
   characterization tests first or say plainly that the change is unverified.
5. **Name the rule.** Cite codes when reviewing or reporting (`G5`, `F3`, `N1`) so the user can
   look them up in [references/catalog.md](references/catalog.md).
6. **Prefer the simplest thing.** No speculative abstractions, unused parameters, or
   "future-proof" layers (Simple Design rule 4, G12).

## Pick a mode

| The user wants...                            | Do this                                                          | Read first                                                                                                                                         |
| -------------------------------------------- | ---------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| New or changed code                          | Apply the checklist below while writing; run the tests           | [references/principles.md](references/principles.md)                                                                                               |
| A review of code or a diff                   | Findings by severity, each with a rule code and a fix            | [references/review.md](references/review.md)                                                                                                       |
| A cleanup or refactor                        | Small verified steps, one smell at a time                        | [references/refactor.md](references/refactor.md)                                                                                                   |
| An explanation of a rule, term, or technique | Answer from the catalog, glossary, or patterns                   | [references/catalog.md](references/catalog.md), [references/glossary.md](references/glossary.md), [references/patterns.md](references/patterns.md) |
| "Any quick wins?" while editing              | Boy Scout pass: one or two safe improvements in the touched code | this file                                                                                                                                          |

If the request is ambiguous, default to **write** when producing code and **review** when
shown existing code. Anything after the skill name (for example `review src/billing`) is the
target; treat it as a path, a rule code, or a topic as appropriate.

## Checklist while writing

**Names**

- Reveal intent; avoid disinformation and noise words; make names pronounceable and searchable (N1, N4).
- Classes and types are nouns, functions are verbs; one word per concept (N3).
- Name length grows with scope (N5); no type or scope encodings unless the language demands them (N6).
- If a name hides a side effect, rename it (N7, G20).

**Functions**

- Do one thing at one level of abstraction; extract until the body reads like prose (G30, G34).
- Few arguments (F1); no flag arguments (F3); no output arguments (F2); no selector arguments (G15).
- A function either does something or answers something, not both (command-query separation).
- Guard clauses over deep nesting; encapsulate conditions in named predicates (G28, G29).
- Replace magic numbers and strings with named constants or enums (G25); delete dead code (F4, G9).

**Comments**

- Explain _why_ or warn of consequences; never narrate _what_ the code already says (C3).
- No history, authorship, or ticket trails in comments (C1); delete stale ones (C2).
- Never commit commented-out code (C5).

**Errors and boundaries**

- Signal failure with the language's idiomatic mechanism and add context; never swallow errors (G4).
- Do not return or pass null where a special-case object, empty collection, or option type fits.
- Wrap third-party APIs behind your own seam so vendor change stays local.
- Test boundaries explicitly (G3, T5); name every assumption (G26).

**Structure**

- Remove duplication first; it is the root of most smells (G5), then fix abstraction levels (G6).
- Small classes with one reason to change (SRP); extend, do not modify (OCP); depend on abstractions (DIP).
- Hide data, expose behavior; do not navigate through strangers (G14, G36).
- Construct dependencies at the edge (`main`, factories, DI) and inject them.
- Prefer polymorphism over repeated type switches (G23).

**Tests**

- Test the behavior you add or change; one concept per test (T1, T3).
- Fast, Independent, Repeatable, Self-validating, Timely (F.I.R.S.T.) (T9).
- Never skip or weaken a test to get green; an ignored test is an open question (T4, G4).
- Keep one command to build and one to test (E1, E2).

## Language adaptation

Detect the language from the files at hand and read the matching file before writing or
reviewing a substantial change. Where a language idiom contradicts a generic rule, the idiom wins.

| Language                | Reference                                                                |
| ----------------------- | ------------------------------------------------------------------------ |
| Python                  | [references/languages/python.md](references/languages/python.md)         |
| TypeScript / JavaScript | [references/languages/typescript.md](references/languages/typescript.md) |
| Java                    | [references/languages/java.md](references/languages/java.md)             |
| Kotlin                  | [references/languages/kotlin.md](references/languages/kotlin.md)         |
| Go                      | [references/languages/go.md](references/languages/go.md)                 |
| Rust                    | [references/languages/rust.md](references/languages/rust.md)             |
| C#                      | [references/languages/csharp.md](references/languages/csharp.md)         |

For any other language, apply the generic checklist and follow that ecosystem's formatter,
linter, and naming conventions.

## Working with AI-generated code

When you are the one generating code, also follow
[references/ai-assisted-coding.md](references/ai-assisted-coding.md): search for existing
helpers before adding new ones, keep diffs small and reviewable, never invent APIs, and never
silence a failing check to finish faster.

## Reporting what you did

End a task with a short note, not a lecture:

- Edits: `Cleaned up: extracted SECONDS_PER_DAY (G25), removed dead branch (G9).`
- Reviews: findings grouped by severity using the format in [references/review.md](references/review.md).
- Skipped work: `Left as is: the 120-line parse() function is out of scope (G30); worth a follow-up.`

## Scope and limits

This is an independent, condensed aid for applying principles popularized by Robert C. Martin.
It restates the named heuristics and frameworks in original words and does not reproduce the
book's text, code, or structure. Read _Clean Code: A Handbook of Agile Software Craftsmanship_
(Prentice Hall, 2008) for the authoritative treatment. Not affiliated with or endorsed by the
author or publisher. See [references/principles.md](references/principles.md) for the
principles and [NOTICE](https://github.com/patbaumgartner/clean-code-skill/blob/main/NOTICE.md)
for credits.
