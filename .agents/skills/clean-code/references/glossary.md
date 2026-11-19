# Glossary

Short, original definitions of the terms used across this skill.

**Abstract Factory** - A creation pattern that hides which concrete type is built, letting one factory contain the only type switch.

**Active Record** - A data structure with persistence methods such as `save` and `find`; keep business rules out of it.

**Adapter** - A wrapper that makes one interface look like another, isolating a boundary.

**Boundary** - Any place where your code meets code you do not control: libraries, services, other teams.

**Boy Scout Rule** - Leave the code cleaner than you found it, on every change.

**Build-Operate-Check** - The three-part shape of a readable test: arrange the data, act, assert.

**Characterization test** - A test that records what existing code does today so it can be refactored safely.

**Code smell** - A surface symptom that hints at a deeper design problem.

**Cohesion** - How strongly a class's methods use its fields; high cohesion means a focused class.

**Command-Query Separation** - A function either changes state or returns information, not both.

**Cross-cutting concern** - Functionality, such as logging or security, that spans many modules.

**Dependency Injection (DI)** - Supplying a component's collaborators from outside instead of letting it construct them.

**Dependency Inversion Principle (DIP)** - High-level code depends on abstractions, not on concrete details.

**DRY (Don't Repeat Yourself)** - Every piece of knowledge has one authoritative representation (G5).

**DTO (Data Transfer Object)** - A data-only object used to move information across a boundary.

**F.I.R.S.T.** - Properties of good tests: Fast, Independent, Repeatable, Self-validating, Timely.

**Feature envy** - A method that uses another object's data more than its own (G14).

**Flag argument** - A boolean parameter that selects between behaviors (F3).

**Hybrid** - A structure that is half object, half data bag; the worst of both.

**Inversion of Control (IoC)** - Moving a secondary responsibility, such as construction, out of an object to an external mechanism.

**LeBlanc's Law** - Later equals never.

**Law of Demeter** - A method should talk only to its own object, its parameters, objects it creates, and its direct parts (G36).

**Learning test** - A test that explores a third-party API and later guards against breaking upgrades.

**Magic number** - A literal whose meaning is not obvious from the code (G25).

**Newspaper metaphor** - A file reads like an article: headline and summary first, details further down.

**Open-Closed Principle (OCP)** - Open for extension, closed for modification.

**Output argument** - A parameter used to return a result by mutation (F2).

**Polymorphism over switch** - Dispatch on type through objects or sum types instead of repeated conditionals (G23).

**Single Responsibility Principle (SRP)** - A module has one reason to change.

**Special case object** - An object that encapsulates a default or exceptional behavior so callers need no checks.

**Stepdown Rule** - Each function is followed by the functions at the next level of abstraction down.

**Successive refinement** - Reaching clean code through many small, test-protected improvements.

**Template Method** - A pattern that hoists a shared algorithm into a base type with overridable steps.

**Temporal coupling** - A hidden requirement that calls happen in a specific order (G31).

**Test-Driven Development (TDD)** - Writing a failing test before the code that makes it pass.

**Three Laws of TDD** - No production code without a failing test; no more test than needed to fail; no more code than needed to pass.

**Train wreck** - A chain such as `a.b().c().d()` that reaches through object internals (G36).

**Vertical distance** - How far apart related lines of code sit in a file (G10).
