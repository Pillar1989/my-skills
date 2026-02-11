---
name: baozhu-python-style
description: Use when creating Python 3.10+ projects or components that need clean architecture, practical type hints, and professional code quality. Generates project structures, abstract base classes, and tests following pragmatic conventions inspired by large-scale open-source Python codebases.
---

# Baozhu Python Style

## Overview

Generate professional Python code with consistent conventions, practical type annotations, and clean architecture. Prioritizes readability and maintainability over dogmatic rules.

## When to Use

- Creating new Python 3.10+ projects from scratch
- Adding well-architected components to existing projects
- Want consistent coding conventions across a codebase

## When NOT to Use

- Quick prototyping or throwaway scripts
- Projects that must support Python < 3.10

## Design Principles

- Consistent formatting across all files, no exceptions
- Flat is better than nested; explicit is better than implicit
- All function parameters and return types must be annotated
- Use Python 3.10+ syntax: `dict`, `list`, `tuple` (lowercase), `X | Y` union, `X | None` for optional
- Module-level type aliases for complex recurring types; reserve `Any` for truly dynamic data with clear justification
- Class-level type annotations for attributes set outside `__init__`
- Define extension points using abstract base classes with `Base*` prefix. Never extend concrete classes
- Keep inheritance flat: one level preferred, two maximum. Prefer composition over deep inheritance

## Formatting and Toolchain

Line width 79 characters. Single quotes for strings. Toolchain config in `setup.cfg`:

```ini
[isort]
line_length = 79
multi_line_output = 0
known_first_party = <your_package>
known_third_party = pytest
no_lines_before = STDLIB,LOCALFOLDER
default_section = THIRDPARTY

[yapf]
BASED_ON_STYLE = pep8
BLANK_LINE_BEFORE_NESTED_CLASS_OR_DEF = true
SPLIT_BEFORE_EXPRESSION_AFTER_OPENING_PAREN = true
```

- **Copyright header**: Every `.py` file starts with `# Copyright (c) Your Organization. All rights reserved.`
- **Inline suppression** — use sparingly, with specific codes: `# noqa: F401`, `# type: ignore[arg-type]`, `# yapf: disable` / `# yapf: enable`

### File Content Order

1. Copyright header → 2. Module docstring → 3. Imports (stdlib → third-party → local, sorted by isort) → 4. Constants/enums (`UPPER_SNAKE`) → 5. Type aliases → 6. Abstract base classes → 7. Concrete classes → 8. Public functions → 9. Private helpers (`_leading_underscore`)

## Docstring Conventions

Google-style, plain text. No markup language or cross-reference syntax.

### Canonical Example

```python
def load_config(
    filepath: str | Path,
    backend_args: dict | None = None,
    encoding: str = 'utf-8',
) -> Config:
    """Load configuration from a file.

    Args:
        filepath (str | Path): Path to the config file.
        backend_args (dict, optional): Arguments for the file backend.
            Defaults to None.
        encoding (str): File encoding. Defaults to 'utf-8'.

    Returns:
        Config: The loaded configuration object.

    Raises:
        FileNotFoundError: If ``filepath`` does not exist.

    Note:
        Supported formats are JSON, YAML, and Python files.

    Examples:
        >>> cfg = load_config('config.yaml')
    """
```

### Key Rules

- **Type** in `(parentheses)` after arg name, using `|` for unions (e.g., `str | Path`). Legacy codebases may use comma-separated `(str, Path)` — pick one style and stay consistent
- **Optional** expressed as `(type, optional):` when default is `None`
- **Defaults** stated as sentence: `Defaults to X.`
- **Continuation lines** indented 4 extra spaces to align with description
- **Backticks** (``` `` `` ```) for inline code references in descriptions
- Use `New in version X.Y.Z.` for API additions
- **Returns**: `type: description.` on one line; for complex returns use sub-list with `- 'key' (type): description.`
- **Property docstrings**: one-line with type prefix: `"""str: The working directory."""`
- **Class docstrings**: constructor `Args:` go in the **class docstring**, not in `__init__`'s docstring. `__init__` has no docstring. This keeps the public API documented in one place

## Type Annotations

### Python 3.10+ Syntax — Mandatory

```python
# Correct — 3.10+ style
def process(data: list[dict[str, int]], callback: Callable[[str], None] | None = None) -> tuple[list[str], dict[str, float]]: ...
# Wrong — do NOT use: List, Dict, Optional, Tuple from typing
```

- **Module-level type aliases** for complex types: `ConfigType = dict | Config | ConfigDict`. Convention: `PascalCase` for object-like aliases, `UPPER_SNAKE` for simple union aliases
- **Class-level type declarations** for attributes set later (not in `__init__` params): bare annotation like `model: nn.Module` in class body

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Classes | PascalCase | `DataProcessor` |
| Abstract bases | `Base*` prefix | `BaseDataset`, `BaseMetric` |
| Functions / methods | snake_case | `process_data` |
| Constants | UPPER_SNAKE | `MAX_RETRIES` |
| Private attributes | leading `_` | `_internal_cache` |
| Type aliases | PascalCase or UPPER_SNAKE | `ConfigType`, `DATA_BATCH` |
| Boolean functions | `is_*` / `has_*` prefix | `is_valid`, `has_permission` |
| Factory functions | `build_*` prefix | `build_optimizer` |
| Alternative constructors | `from_*` classmethod | `from_cfg`, `from_dict` |

### File Naming

- **Module files**: `snake_case.py` — one main class per file (`checkpoint_hook.py` → `CheckpointHook`)
- **Abstraction files**: `base.py`, `types.py`, `constants.py`
- **Utility files**: `misc.py`, `helpers.py`, `decorators.py`
- **Test files**: `test_` prefix mirroring source: `test_checkpoint_hook.py`

### Import Organization

Three groups, sorted alphabetically within each, enforced by isort: 1) Standard library 2) Third-party 3) Local. In `__init__.py`, use explicit imports with `__all__`. Top-level `__init__.py` may re-export sub-packages with `# flake8: noqa` and `from .subpkg import *`.

## Project Layout

```
project-name/
├── project_name/                 # Main package at repo root (NO src/)
│   ├── __init__.py               # Re-exports with __all__
│   ├── version.py
│   ├── config/
│   │   ├── __init__.py           # Explicit __all__ exports
│   │   ├── config.py
│   │   └── utils.py
│   ├── hooks/
│   │   ├── __init__.py
│   │   ├── hook.py               # Base class
│   │   └── checkpoint_hook.py    # Concrete class
│   └── utils/
│       ├── __init__.py
│       └── misc.py
├── tests/                        # Mirror package structure
│   ├── conftest.py
│   ├── test_hooks/
│   │   ├── test_hook.py
│   │   └── test_checkpoint_hook.py
│   └── test_config/
├── requirements/                 # Split dependency files
│   ├── runtime.txt
│   ├── tests.txt
│   └── docs.txt
├── setup.py
├── setup.cfg                     # Tool configurations
├── README.md
└── LICENSE
```

- **No `src/` directory** — package lives at repo root
- **One main class per file**, file named after the class in snake_case
- **Tests mirror package structure** with `test_` prefix directories
- **Dependencies split** into `requirements/` with purpose-specific files
- Every `__init__.py` has explicit `__all__`

## Class Design Patterns

### Class-Level Constants and Defaults

Define class-level constants, default lookup tables, and type declarations **between the class declaration and `__init__`**. This is the standard place for shared configuration that subclasses may override:

```python
class CheckpointHook(BaseHook):
    """Save checkpoints periodically.

    Args:
        interval (int): Saving period. Defaults to -1.
    """

    out_dir: str                    # type declaration (set later)
    priority = 'VERY_LOW'          # public constant, subclass may override
    rule_map = {'greater': lambda x, y: x > y,
                'less': lambda x, y: x < y}
    _default_greater_keys = ['acc', 'top']  # private default

    def __init__(self, interval: int = -1) -> None: ...
```

- **Public constants** (`priority`, `rule_map`): shared behavior, subclasses may override
- **Private defaults** (`_default_greater_keys`): internal defaults, prefixed with `_`
- **Type declarations** (`out_dir: str`): attributes set later in lifecycle, not in `__init__`

### Private Attribute + Property Encapsulation

The default pattern for exposing internal state. Two variants:

**Simple getter** — `self._work_dir` stored in `__init__`, exposed via `@property` returning `self._work_dir`. Property docstring: `"""str: The working directory."""`

**Lazy initialization** — `self._train_loop` stores raw config (dict), `@property` checks `isinstance(self._train_loop, dict)` and builds on first access, then caches.

### `__init__` Method Structure

Follow four phases in order:

```python
def __init__(self, interval: int = -1, by_epoch: bool = True,
             save_best: str | list[str] | None = None,
             backend_args: dict | None = None,
             file_client_args: dict | None = None,  # deprecated
             **kwargs) -> None:
    # Phase 1: Direct attribute storage
    self.interval = interval
    self.by_epoch = by_epoch

    # Phase 2: Deprecation compatibility
    if file_client_args is not None:
        warnings.warn('"file_client_args" is deprecated, use "backend_args"',
                       DeprecationWarning, stacklevel=2)
        if backend_args is not None:
            raise ValueError('"file_client_args" and "backend_args" cannot both be set')
        self.backend_args = file_client_args
    else:
        self.backend_args = backend_args

    # Phase 3: Default value derivation
    if save_best is None:
        self.save_best = []
    elif isinstance(save_best, str):
        self.save_best = [save_best]
    else:
        self.save_best = save_best

    # Phase 4: Validation with assertions
    assert isinstance(self.interval, int), (
        f'"interval" must be an int, got {type(self.interval)}')

    # Optional: capture extra kwargs for extensible classes
    self.args = kwargs
```

- Phase order is strict: storage → deprecation → derivation → validation
- `**kwargs` capture is common for extensible classes where subclasses or configs may pass extra parameters
- `__init__` has no docstring — all `Args:` are documented in the class docstring

### Alternative Constructors with `@classmethod`

Use `from_*` naming for building from different sources. Method calls `copy.deepcopy(cfg)` then delegates to `cls(...)`. Examples: `from_cfg`, `from_dict`, `from_file`.

### Named Singleton (Manager Mixin)

For classes that need global access by name (loggers, message hubs, visualizers). Core API:

- `_instance_dict: dict[str, 'ManagerMixin'] = {}` class-level dict
- `__init__(self, name: str, **kwargs)` — asserts `name` is non-empty string, stores as `self._instance_name`
- `get_instance(cls, name, **kwargs)` classmethod — get or create named instance
- `get_current_instance(cls)` classmethod — returns most recently created instance, raises `RuntimeError` if none
- `instance_name` property — read-only access to `self._instance_name`

### Context Manager Pattern

Use `@contextmanager` for temporary state changes. Pattern: save old state → set new state → `try: yield` → `finally: restore`. Handle `None` input with early `yield` + `return`.

## Error Handling

### Assert vs Raise — Clear Division

**`assert`** for internal invariants and developer-facing preconditions. Always include message string with parenthesized tuple format:

```python
assert isinstance(name, str) and name, (
    'name argument must be a non-empty string.')
assert not (begin_iter != 0 and begin_epoch != 0), (
    '`begin_iter` and `begin_epoch` should not both be set.')
```

**`raise`** for user-facing errors. Always with f-string context showing actual values:

```python
if not isinstance(imports, list):
    raise TypeError(f'custom_imports must be a list but got {type(imports)}')
if not force and name in self._module_dict:
    raise KeyError(f'{name} is already registered in {self.name} at {self._module_dict[name].__module__}')
```

**`warnings.warn`** for deprecations and non-fatal issues. Always include `DeprecationWarning` category and `stacklevel=2`.

### Custom Exception Hierarchy

Project-level base exception (`ProjectError(Exception)`) with specific subclasses (`ConfigError`, `BuildError`).

### Deprecation Decorators

`deprecated_function(since, removed_in, instructions)` → decorator that wraps with `warnings.warn(... DeprecationWarning, stacklevel=2)` and delegates to original function via `@functools.wraps`.

## Logging Convention

Use a centralized `print_log(msg, logger, level)` function instead of scattering `logging.getLogger()` calls. Provides a single point to control log format, destination, and level:

- `logger=None` → `print(msg)` to stdout
- `logger='name'` (str) → `logging.getLogger(name).log(level, msg)`
- `logger=<Logger instance>` → `logger.log(level, msg)`

Usage: `print_log(f'Checkpoint saved to {filepath}', logger='current')`. Use `level=logging.WARNING` for deprecation warnings via logging.

## Testing

- **pytest** as test framework with fixtures in `conftest.py`
- **Arrange-Act-Assert** pattern in every test method
- **`TestClassName`** with `test_method_describes_behavior` naming
- Test directories mirror package structure: `tests/test_hooks/test_checkpoint_hook.py`
- Clean up global state (singletons, registries) in fixtures or teardown
- Use `tmp_path` fixture for temporary working directories
- Mock at the boundary of the unit under test, prefer dependency injection

## Optional Design Patterns

These patterns are **not mandatory**. Choose based on actual project needs.

### Registry Pattern

**When to use**: Plugin architectures where components are selected by name at runtime from config.

A global `Registry` object maps string names to classes. Core API: `__init__(name)` with `_module_dict: dict[str, type]`, `register_module(module, name, force)` usable as decorator or function call (raises `KeyError` if name exists and `force=False`), `build(cfg)` pops `type` key from config dict and instantiates. Define registries as module-level singletons: `MODELS = Registry('model')`. Register via `@MODELS.register_module()`. Build from config: `MODELS.build({'type': 'ResNet', 'depth': 50})`.

### Hook / Callback Lifecycle

**When to use**: Extensible pipelines where multiple concerns need to observe the same process without tight coupling.

Base hook defines all lifecycle points as no-op methods (`before_run`, `after_run`, `before_epoch`, `after_epoch`, `before_iter`, `after_iter`). Concrete hooks override only what they need. A `priority` class attribute (`HIGHEST` / `HIGH` / `NORMAL` / `LOW` / `LOWEST`) controls execution order. Base methods use **docstrings as the method body** (no `pass`) — a method with only a docstring is valid Python and serves as both documentation and a no-op.

### Hierarchical Configuration

**When to use**: Projects with complex config trees where common settings are shared across variants.

Config files inherit from base configs via `_base_` key. Child configs override or extend parent values.

### Decorator Guard

**When to use**: Methods that should only execute under certain conditions (e.g., main process only, after initialization).

`master_only(func)` — only execute on rank 0, return `None` otherwise. `require_initialized(func)` — check `self._initialized`, auto-call `self.full_init()` with warning if not set. Both use `@functools.wraps`.

## Common Mistakes to Avoid

| Mistake | Correction |
|---------|-----------|
| Omitting type annotations or using `Any` excessively | Annotate all params and returns. `Any` only with justifying comment |
| Subclassing a concrete class as extension point | Extract `Base*` abstract class, both old and new extend it |
| 3+ level inheritance chains | Prefer composition. Mixins for cross-cutting. Max one level (base→concrete) |
| `raise ValueError('invalid')` with no detail | Include actual values: `f'interval must be >= 0, got {interval}'` |
| Missing `__all__` in `__init__.py` | Every `__init__.py` must have explicit `__all__` |
| Tests that verify private methods or internal state | Test public behavior only. Refactoring internals should not break tests |

## Red Flags — STOP and Rethink

- Skipping `__all__` exports — "I'll add them later"
- Extending concrete classes — "just this once"
- `Any` everywhere — avoiding real type thinking
- 3+ level inheritance chains
- `__init__` longer than 50 lines without clear phases
- Bare `assert` without a message string
- `raise Exception(...)` instead of a specific type
- No `@property` encapsulation for complex state
- Copy-pasting code instead of extracting a `Base*` class
- Functions longer than 50 lines

Any of these means: Stop. Rethink the design. Apply this skill's patterns.
