# Java

Apply alongside [catalog.md](../catalog.md). Java is the home of the original heuristics, but
modern Java and its tooling change some of the advice.

## Tooling (E1, E2, G24)

- Use the project's wrapper (`./mvnw` or `./gradlew`) so one command builds and one runs tests
  (`./mvnw verify`, `./gradlew build`).
- Let the project's formatter and linter decide style (Spotless, Checkstyle, google-java-format,
  SpotBugs, Error Prone). Do not hand-format.
- Test with JUnit 5; use AssertJ or the project's assertion library; keep unit tests free of
  I/O (T9).

## Language-specific rules

| Code | Rule | Why |
| --- | --- | --- |
| JV1 | Let the IDE and formatter manage imports; follow the project's wildcard policy (J1) | Many modern style guides forbid wildcards; consistency beats either extreme |
| JV2 | Do not implement an interface just to inherit its constants (J2) | Use static imports or a dedicated type |
| JV3 | Use enums (or sealed types) instead of integer or string constants (J3) | Closed sets get compiler help (G25) |
| JV4 | Return `Optional` for "maybe absent" results; do not use it for fields or parameters | Avoids null returns without cluttering APIs |
| JV5 | Use try-with-resources for anything `AutoCloseable` | Reliable cleanup |
| JV6 | Use `record` for immutable data carriers | Replaces boilerplate DTOs (G12) |
| JV7 | Use sealed interfaces with `switch` patterns for variants | Type-safe G23 |
| JV8 | Prefer constructor injection and `final` fields | Explicit dependencies (G22); testable (DI) |
| JV9 | Catch specific exceptions; wrap with cause; never `catch (Exception e) {}` | Context and no swallowed failures (G4) |

## Idiom notes

- Checked versus unchecked: prefer unchecked exceptions for programming and unrecoverable errors;
  use checked ones only where callers can realistically recover. Do not leak them through layers.
- Avoid `I`-prefixes on interfaces and `Impl` suffixes without cause (N6); name the concept.
- In Spring or other frameworks, keep controllers thin, put logic in services, and keep
  persistence entities out of API responses (DTOs at boundaries).
- Streams are good for simple pipelines; extract a method when the lambda needs a comment.

## Before and after

```java
// before
public void send(String to, boolean urgent) { /* ... */ }
if (customer.getAddress().getCountry().getCode().equals("CH")) { /* ... */ }
```

```java
// after
public void sendNormal(EmailAddress to) { /* ... */ }
public void sendUrgent(EmailAddress to) { /* ... */ }
if (customer.isSwiss()) { /* ... */ }
```

Fixes: F3, G36, G28.
