# Python

Apply alongside [catalog.md](../catalog.md). Where a rule below conflicts with the generic
checklist, the Python idiom wins.

## Tooling (E1, E2, G24)

- Format and lint with the project's tools; common choices are `ruff` (lint and format) or
  `black` plus `flake8`, with `mypy` or `pyright` for types.
- Test with `pytest` (or the project's runner); measure coverage with `pytest --cov`.
- Prefer one command to install (`pip install -e ".[dev]"`, `uv sync`, or `poetry install`) and one
  to test (`pytest`).
- Follow PEP 8 and PEP 257 unless the project documents otherwise.

## Language-specific rules

| Code | Rule | Why |
| --- | --- | --- |
| PY1 | No wildcard imports (`from x import *`) | Hides where names come from (G16, G22); PEP 8 discourages it (the reverse of the Java J1 advice) |
| PY2 | Use `Enum`, `Literal`, or `StrEnum` for closed sets, not magic strings | Same intent as J3 and G25 |
| PY3 | Type-hint public functions, methods, and dataclass fields | Python's equivalent of declared types; makes intent checkable |
| PY4 | Never use mutable default arguments; default to `None` and build inside | A classic hidden-state bug (G26) |
| PY5 | Use context managers (`with`) for files, locks, and connections | Guarantees cleanup without try/finally noise |
| PY6 | Catch specific exceptions; never a bare `except:` | A bare catch swallows interrupts and bugs (G4) |
| PY7 | Group related arguments with a `dataclass` or `NamedTuple` | Satisfies F1 with a named concept |
| PY8 | Make boolean parameters keyword-only (`def f(x, *, strict=False)`) when a flag is truly unavoidable | Softens F3 at the call site; prefer two functions |
| PY9 | Use comprehensions and generators for simple transforms; use a loop when logic grows | Readability over cleverness (G16) |

## Idiom notes

- Docstrings document public behavior (what, arguments, raises); they are not C3 redundancy
  when a tool consumes them. Keep them short and drop ones that restate the signature.
- Truthiness checks (`if items:`) are idiomatic; avoid `if len(items) > 0` noise.
- Prefer `pathlib`, `dataclasses`, `enum`, and the standard library over homegrown helpers (G5).
- Underscore-prefixed names mark non-public members; this is a convention that complements, and
  does not violate, N6.

## Before and after

```python
# before
def proc(d, t, flag=False):  # process data
    x = []
    for i in d:
        if flag:
            if i["type"] == "A":
                x.append(i["val"] * 1.0825)
    return x
```

```python
# after
CA_TAX_RATE = 0.0825

def taxed_values(transactions: list[Transaction]) -> list[float]:
    return [t.value * (1 + CA_TAX_RATE) for t in transactions if t.is_taxable]

def raw_values(transactions: list[Transaction]) -> list[float]:
    return [t.value for t in transactions]
```

Fixes: N1, C3, F3, G25, G23, PY3.
