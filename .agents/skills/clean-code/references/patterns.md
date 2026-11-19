# Patterns and Techniques

Concrete moves in *when / how / trade-offs* form. Code samples are illustrative Python or
TypeScript; the ideas are language-neutral.

## Express intent in code, not comments

**When:** you are about to write a comment that explains a condition or a block.
**How:** extract it into a named function or variable.

```python
# before
if employee.flags & HOURLY and employee.age > 65:  # eligible for full benefits
    ...

# after
if employee.is_eligible_for_full_benefits():
    ...
```

**Trade-off:** one more named unit; names do not drift out of date the way comments do.

## Extract until you drop

**When:** a function does more than one thing, mixes abstraction levels, or has sections.
**How:** extract each step into a function named for *what* it achieves; order callers above
callees (Stepdown Rule).
**Trade-off:** more small functions; navigation relies on good names and the IDE.

## Guard clauses instead of nesting

**When:** the happy path is buried under `if` pyramids.
**How:** handle invalid or trivial cases first and return early.

```python
def shipping_cost(order):
    if order.is_empty():
        return 0
    if order.is_pickup():
        return 0
    return base_rate(order) + weight_surcharge(order)
```

**Trade-off:** multiple return points; acceptable in short functions and clearer than nesting.

## Replace flag arguments with separate functions

**When:** a boolean or enum parameter switches behavior (F3, G15).
**How:** split into two honestly named functions, or into polymorphic objects.

```python
render(page, True)          # before: what does True mean?
render_for_print(page)      # after
render_for_screen(page)
```

**Trade-off:** more API surface; each function becomes obvious and independently testable.

## Parameter object

**When:** several arguments travel together through many calls (F1).
**How:** introduce a small immutable type for the group and give it behavior where natural.
**Trade-off:** one more type; call sites shrink and the concept gets a name.

## Replace switch with polymorphism (Abstract Factory)

**When:** the same type switch appears in more than one place (G23).
**How:** hide the single unavoidable switch in a factory that returns polymorphic objects; add
new cases by adding classes (OCP). In languages with sum types, use a closed union with
exhaustive matching instead.
**Trade-off:** more types; repeated, error-prone branching disappears.

## Special case object

**When:** callers keep checking for null or catching an exception to apply a default.
**How:** return an object that implements the same interface and encapsulates the default
behavior (for example `NullCustomer`, `GuestSession`, an empty collection).
**Trade-off:** one extra class; client code loses its conditionals.

## Wrap third-party APIs

**When:** a library type or exception would otherwise spread through the codebase.
**How:** expose only the operations you need in your own vocabulary; translate the vendor's
exceptions.
**Trade-off:** a thin layer; gains testability and a single place to migrate.

## Learning tests

**When:** adopting or upgrading a library.
**How:** write tests that exercise the API as you intend to use it; keep them in the suite.
**Trade-off:** small upfront effort that you would spend exploring anyway; catches breaking
upgrades for free.

## Code to the interface you wish you had

**When:** you depend on something that does not exist yet (another team's API).
**How:** define the ideal interface, build against it with a fake, adapt reality later.
**Trade-off:** an Adapter to write later; your design stays clean and testable.

## Separate construction from use

**When:** objects build their own collaborators, or business logic contains `new`.
**How:** move wiring to `main`, factories, or a DI container; pass complete collaborators in;
depend on interfaces.
**Trade-off:** a wiring layer; yields swappable, testable components.

## Extract class (apply SRP)

**When:** a class has methods that use disjoint subsets of its fields, or a name that needs
"and".
**How:** move the cohesive subset and its fields to a new class named for the responsibility.
**Trade-off:** more classes; each is smaller and has one reason to change.

## Characterization test before refactoring

**When:** legacy code has no tests but must change safely.
**How:** call it with representative inputs, record what it actually does, assert that, then
refactor while the tests stay green.
**Trade-off:** pins current behavior including bugs; fix bugs separately, deliberately.

## Introduce an explanatory variable

**When:** a long expression hides meaning (G19).
**How:** name each meaningful sub-expression.
**Trade-off:** a few extra lines; the code reads like the domain.

## Encapsulate the condition

**When:** a boolean expression spans several clauses or appears twice (G28).
**How:** extract a predicate function with a positive name (G29).

## Concurrency defenses

**When:** introducing threads, tasks, or async pipelines.
**How:** isolate concurrent code; limit and protect shared mutable state; prefer immutable data
and message passing; use standard primitives; make failures reproducible by varying timing.
**Trade-off:** extra structure; intermittent bugs become tractable.

## Build a domain-specific testing language

**When:** tests are long, noisy, or hard to read.
**How:** extract builders and assertion helpers so a test reads as a statement about the domain.

```python
order = an_order().with_items(2).shipped_to("CH").build()
assert_that(order).costs(12_00).and_ships_in_days(3)
```

**Trade-off:** a small test API to maintain; tests become short and durable.

## Successive refinement under tests

**When:** a change starts to sprawl, or the code is sliding toward a mess.
**How:** stop adding features; refactor in tiny steps with the suite green; resume afterward.
**Trade-off:** a short pause now avoids an unrecoverable mess later.
