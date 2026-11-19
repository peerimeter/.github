# Principles

The core frameworks, grouped by topic. All wording is original and condensed. Use it to decide
*why* a change is worth making; use [catalog.md](catalog.md) to name *what* is wrong.

## Prime directive

Clean code does one thing well and reads like well-written prose. The only way to go fast is
to keep the code clean, so cleanliness is part of the work, not a bonus. Apply the **Boy Scout
Rule**: leave every module a little better than you found it. Beware of **LeBlanc's Law**:
later equals never, so do the small cleanup now.

**When principles conflict, decide in this order:**

1. Correctness and passing tests.
2. Project conventions and the surrounding code.
3. Removing duplication (G5) and expressing intent (G16).
4. Size and structure (small functions, small classes).
5. Cosmetic preferences.

## Names

- A name should tell why something exists, what it does, and how it is used. If it needs a
  comment, it failed.
- Avoid names that lie (`accountList` that is not a list), noise words (`Info`, `Data`,
  `Manager`) that add nothing, and near-identical names that differ by a character.
- Make names pronounceable and searchable; single letters are only for tiny local scopes.
- Classes and types are nouns; functions are verbs; booleans read as questions
  (`isExpired`, `hasAccess`).
- One word per concept across the codebase (`fetch` or `get`, not both). Avoid puns where one
  word carries two meanings.
- Use domain vocabulary for domain concepts and technical vocabulary for technical ones.

## Functions

- **Small, then smaller.** A function that fits in a glance can be understood without scrolling.
- **Do one thing**, at **one level of abstraction**. If you can extract a function whose name is
  not a restatement of its body, the original did more than one thing.
- **Stepdown Rule:** read top-down; each function is followed by those one level below it.
- **Arguments:** zero is best, then one, then two; three needs a reason. Booleans as arguments
  announce that the function does two things. Do not use parameters for output.
- **Command-query separation:** either change state or return information, not both.
- **No hidden side effects.** If a function does something its name does not mention, fix the
  name or the function.
- **Prefer exceptions (or the language's error type) over status codes**, and extract the
  try/catch body so error handling is its own function.
- **DRY.** Duplication is the root of most evil: each fix and each change must be made once.

## Comments

- Code is the primary way to express intent; a comment is a last resort for what code cannot say.
- Good comments: legal notices, intent behind a surprising choice, clarification of an
  unchangeable external API, warnings of consequences, TODOs that name an owner or issue,
  public API documentation that a tool consumes.
- Bad comments: mumbling, redundancy, journals, closing-brace markers, banners, attribution,
  and above all commented-out code.
- Comments rot because nothing checks them. Prefer to delete and rename.

## Formatting

- Formatting is communication. Adopt one team style, enforce it with a formatter, and stop
  debating it.
- Read like a newspaper: high-level concepts first, details below.
- Blank lines separate concepts; related lines stay together (vertical density).
- Declare variables near their use; keep instance variables in one predictable place.
- Keep lines short enough to read without horizontal scrolling.

## Objects and data structures

- **Objects** hide their data and expose behavior. **Data structures** expose their data and have
  no meaningful behavior. They are opposites, not variations.
- Procedural code makes adding new functions easy and new types hard. Object-oriented code is the
  reverse. Pick based on which axis is more likely to change.
- **Avoid hybrids** that are half object, half data bag.
- **Law of Demeter:** a method talks to its own object, its parameters, objects it creates, and
  its direct components, never to the internals of what those return. Avoid train wrecks.
- **Tell, don't ask.** Data-transfer objects belong at boundaries (I/O, persistence, APIs).

## Error handling

- Error handling is one thing; a function that handles errors should do little else.
- Throw or return errors with enough context to locate and understand the failure: the operation,
  the inputs that matter, and the cause.
- Do not return null to signal "nothing", and do not pass null as an argument. Use empty
  collections, option types, or a **special case object** that carries the default behavior.
- Wrap third-party error types so callers depend on your abstraction, not the vendor's.
- Never swallow errors silently.

## Boundaries

- Third-party and cross-team code should enter your system through a narrow seam you own.
- **Learning tests** pin down how a library behaves and later detect breaking upgrades.
- For code that does not exist yet, design the interface you wish you had, then adapt to
  reality with an **Adapter**.

## Tests

- **Three Laws of TDD:** write no production code until a failing test demands it; write no more
  of a test than is sufficient to fail; write no more production code than is sufficient to pass.
- **Tests are first-class code.** Dirty tests rot, and rotten tests are abandoned, which takes
  away your freedom to change the code.
- Readability is paramount: build, operate, check. A small test-specific language of helpers
  and builders often beats raw setup.
- **One concept per test**; the number of asserts matters less than a single focus.
- **F.I.R.S.T.:** Fast, Independent, Repeatable (any environment), Self-validating (pass or
  fail, no manual inspection), Timely (written with or before the code).

## Classes

- Classes are small in **responsibilities**, not just lines. Name them; if you cannot do so
  concisely, they are doing too much.
- **Single Responsibility Principle:** one reason to change. Group things that change together;
  separate those that change for different reasons.
- **Cohesion:** methods should use the instance variables. When a subset of methods uses a subset
  of fields, that subset wants to be its own class.
- **Open-Closed Principle:** extend by adding code, not by editing proven code.
- **Dependency Inversion Principle:** depend on abstractions, not concretions, so details can be
  swapped and tested.
- Organize for change: isolate what varies behind interfaces.

## Systems

- **Separate construction from use.** Build objects and wire them at the edge (`main`,
  factories, DI containers); business logic receives finished collaborators.
- Lazy initialization inside logic buries hard-coded dependencies and breaks testing.
- **Dependency injection** is inversion of control applied to construction.
- Grow architecture incrementally: keep concerns separated so that decisions can wait until the
  **last responsible moment**, when the most information is available.
- Cross-cutting concerns (security, logging, transactions) belong in a mechanism designed for
  them (aspects, middleware, decorators), not scattered through the domain code.
- Standardize and use domain-specific language where it makes intent visible.

## Emergent design: the four rules of simple design

In priority order, a design is simple when it:

1. Runs all the tests.
2. Contains no duplication.
3. Expresses the intent of the programmer.
4. Minimizes the number of classes and methods.

Tests make refactoring safe, which lets rules 2 to 4 be applied continuously.

## Concurrency

- Concurrency decouples *what* from *when*, at the price of complexity and non-determinism.
- Keep concurrency code separate (SRP); it has its own life cycle and failure modes.
- Limit the scope of shared mutable data and protect what remains; prefer copies and
  confinement; keep critical sections small.
- Know your library's concurrency primitives and prefer them to hand-rolled locks.
- Deadlock needs four conditions at once: mutual exclusion, hold and wait, no preemption, and
  circular wait. Break any one.
- Treat intermittent failures as threading bugs until proven otherwise; make threaded code
  testable and exercise it under varied load and scheduling.

## Successive refinement and definition of done

Clean code is rarely written clean the first time. Get it working, then refine in small
test-protected steps. Stop adding features the moment the code starts to rot, and clean first.
Work is done when it **works** (tested) and it is **right** (clean).
