# Smell and Heuristic Catalog

Every named heuristic with a one-line symptom and the usual remedy. Cite codes in reviews and
change notes. The codes are standard terminology in the clean-code community; the wording here
is original. Language-specific variants live in [languages/](languages/).

Most messes reduce to two root causes: **G5 duplication** and **G6 wrong level of
abstraction**. When unsure where to start, start there.

## Comments (C)

| Code | Heuristic | Look for | Do instead |
| --- | --- | --- | --- |
| C1 | Inappropriate information | Author, date, ticket, or change-log text in comments | Let version control and the tracker hold history |
| C2 | Obsolete comment | A comment describing behavior that no longer exists | Update or delete it the moment you notice |
| C3 | Redundant comment | A comment that repeats what the code plainly says | Delete it; improve the name instead |
| C4 | Poorly written comment | Rambling, vague, or misleading explanation | If it earns its place, make it short, exact, and about *why* |
| C5 | Commented-out code | Dead code kept "just in case" | Delete it; history remembers |

## Environment (E)

| Code | Heuristic | Look for | Do instead |
| --- | --- | --- | --- |
| E1 | Build needs many steps | Manual setup, hidden tools, multi-step builds | One documented command builds everything |
| E2 | Tests need many steps | Several commands or manual prep to run the tests | One command runs all tests |

## Functions (F)

| Code | Heuristic | Look for | Do instead |
| --- | --- | --- | --- |
| F1 | Too many arguments | More than about three parameters | Group related ones into a type; split the function |
| F2 | Output arguments | Parameters mutated to hand results back | Return the value, or change the state of the owning object |
| F3 | Flag arguments | A boolean that switches the function's behavior | Two functions with honest names |
| F4 | Dead function | A function nothing calls | Delete it |

## General (G)

| Code | Heuristic | Look for | Do instead |
| --- | --- | --- | --- |
| G1 | Multiple languages in one file | HTML, SQL, JS, and regex tangled into one source file | Separate files, templates, or builders; keep each file one language where possible |
| G2 | Obvious behavior unimplemented | A function missing the behavior its name promises | Implement the least-surprising behavior |
| G3 | Incorrect behavior at boundaries | Edge cases handled by intuition | Test and handle every boundary explicitly |
| G4 | Overridden safeties | Muted warnings, disabled linters, skipped or weakened tests | Restore the safety net and fix the cause |
| G5 | Duplication | Copy-paste, near-identical branches, repeated switch chains | Extract a function, template method, polymorphism, or data table |
| G6 | Code at the wrong level of abstraction | Low-level detail in high-level code, or the reverse | Move it to the layer where it belongs |
| G7 | Base class depends on its derivatives | A parent that names or checks its subclasses | Invert the dependency or restructure |
| G8 | Too much information | Wide public API, many globals or exposed fields | Shrink the surface; hide what clients do not need |
| G9 | Dead code | Unreachable branches, unused variables and parameters | Delete it |
| G10 | Vertical separation | Variables or helpers declared far from where they are used | Declare near first use; place callees just below callers |
| G11 | Inconsistency | One concept handled several ways; naming that drifts | Pick the existing convention and apply it everywhere |
| G12 | Clutter | Unused defaults, constructors, imports, noise | Remove it |
| G13 | Artificial coupling | Unrelated things tied together by placement or shared constants | Move each to where it is actually used |
| G14 | Feature envy | A method more interested in another object's data than its own | Move the method to the object whose data it uses |
| G15 | Selector arguments | An enum, int, or boolean picks which behavior runs | Split into separate functions or use polymorphism |
| G16 | Obscured intent | Dense one-liners, cryptic operators, magic values | Rewrite so the intent is obvious |
| G17 | Misplaced responsibility | Code that lives where a reader would not look for it | Place it by least surprise |
| G18 | Inappropriate static | A static function that might one day need to vary | Prefer an instance method unless it can never vary |
| G19 | Missing explanatory variables | A complex expression with no named parts | Name the intermediate results |
| G20 | Function name does not say what it does | A name that hides an effect or a cost | Rename it, or split the function |
| G21 | Algorithm not understood | Code that passes tests after trial-and-error tweaks | Understand it, then refactor until correctness is evident |
| G22 | Logical dependency not physical | A module assumes a fact owned elsewhere (a size, a format) | Ask for the fact explicitly via parameter or interface |
| G23 | Prefer polymorphism to conditionals | The same type switch or if/else chain repeated | One switch in a factory; dispatch polymorphically |
| G24 | Standard conventions ignored | Team or language style broken | Adopt a formatter and linter and run them |
| G25 | Magic numbers | Raw literals with hidden meaning | Named constants or enums |
| G26 | Imprecision | Ignored null, race, rounding, or ambiguity; floats for money | Decide explicitly: decimals, locks, checks, documented limits |
| G27 | Structure over convention | A rule that only works if everyone remembers it | Enforce it with types, structure, or tooling |
| G28 | Unencapsulated conditional | A long boolean expression inline | A named predicate function |
| G29 | Negative conditionals | `if (!isNotReady)` style logic | Express the positive form |
| G30 | Function does more than one thing | Sections or comments separating phases inside a function | Extract each phase |
| G31 | Hidden temporal coupling | Calls that must happen in order with nothing enforcing it | Make each step's output the next step's input |
| G32 | Arbitrary structure | Layout or placement chosen for no reason | Have a reason, and let the structure show it |
| G33 | Unencapsulated boundary condition | `index + 1` or `length - 1` scattered around | Compute once into a named variable |
| G34 | Mixed abstraction levels in a function | High-level steps next to low-level detail | Extract so each function stays on one level |
| G35 | Configuration buried low | Defaults and constants hidden in deep code | Surface them at a high level and pass them down |
| G36 | Transitive navigation | `a.b().c().d()` chains reaching through objects | Tell, don't ask; give the first object a method for it |

## Names (N)

| Code | Heuristic | Look for | Do instead |
| --- | --- | --- | --- |
| N1 | Non-descriptive names | `data`, `tmp`, `doIt`, `x` outside tiny scopes | Pick names that reveal intent |
| N2 | Wrong abstraction level in a name | An interface named after one implementation detail | Name the concept, not the mechanism |
| N3 | Nonstandard nomenclature | Reinventing established terms | Use pattern, domain, and ecosystem vocabulary |
| N4 | Ambiguous names | A name that could mean several things | Be specific and unambiguous |
| N5 | Name length does not match scope | Long names in tiny scopes, cryptic names in wide ones | Short for small scopes, descriptive for large |
| N6 | Encodings | Hungarian notation, `m_` or type prefixes | Drop them unless the language or ecosystem requires them |
| N7 | Names hide side effects | `getConnection()` that creates one | Name the whole behavior, e.g. `createOrReuseConnection()` |

## Tests (T)

| Code | Heuristic | Look for | Do instead |
| --- | --- | --- | --- |
| T1 | Insufficient tests | Behavior that could break with no test | Test everything that could plausibly break |
| T2 | No coverage tool | Coverage never measured | Use a coverage tool as a guide, not a goal |
| T3 | Skipping trivial tests | "Too simple to test" logic | Write them; they are cheap and document behavior |
| T4 | Ignored test | `skip` or `@Disabled` with no reason | Treat it as an open question; fix, delete, or explain |
| T5 | Boundaries untested | Only the happy path covered | Test empty, one, many, first, last, off-by-one, limits |
| T6 | Not exhaustive near bugs | A bug fixed with a single test | Test the neighboring cases; bugs cluster |
| T7 | Failure patterns unexamined | Several tests failing alike | Look for the shared cause |
| T8 | Coverage patterns ignored | Untested paths that share a trait | Ask why those paths are hard to reach |
| T9 | Slow tests | Tests that hit disks, networks, or sleeps | Fake the slow parts; keep unit tests fast |

## Language-specific heuristics

The original Java-flavored trio, restated generically:

| Code | Heuristic | Generic reading |
| --- | --- | --- |
| J1 | Import lists | Keep import style consistent with the ecosystem's tooling; do not hand-maintain it |
| J2 | Do not inherit constants | Reference constants from where they live; never inherit just to reach them |
| J3 | Constants versus enums | Use enums or literal unions for closed sets of values |

See [languages/](languages/) for the per-language equivalents (for example `PY1`, `TS3`, `GO2`).
