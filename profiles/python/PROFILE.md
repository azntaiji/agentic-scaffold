# Profile: Python

## Toolchain

- Python 3.11 or later. State the floor in `pyproject.toml` `requires-python`.
- Virtual environment at `.venv`. Always invoke tools as `.venv/bin/python -m <tool>` so
  the right interpreter is used whatever the shell's state.
- `ruff` for formatting and linting, `pytest` for tests, `mypy` for type checking.
- Dev tools are declared under `[project.optional-dependencies] dev` in `pyproject.toml`.

## Commands

Copy into the Commands table in `AGENTS.md`.

| Task | Command |
|---|---|
| Setup | `python3 -m venv .venv && .venv/bin/python -m pip install -e ".[dev]"` |
| Format | `.venv/bin/python -m ruff format .` |
| Lint | `.venv/bin/python -m ruff check .` |
| Typecheck | `.venv/bin/python -m mypy src` |
| Test (all) | `.venv/bin/python -m pytest -q` |
| Test (single) | `.venv/bin/python -m pytest path/to/test_file.py::test_name -q` |
| Build | `.venv/bin/python -m build` |

## Layout

`src/<package_name>/` for code, `tests/` for tests, `pyproject.toml` for all tool
configuration. Use the `src` layout so tests run against the installed package.

## Versioning

`VERSION` holds SemVer. Python packaging requires PEP 440, which writes pre-releases
differently, so the mirrors use the PEP 440 form and the scaffold check converts.

| `VERSION` (SemVer) | `pyproject.toml` and `__version__` (PEP 440) |
|---|---|
| `1.2.0` | `1.2.0` |
| `1.2.0-alpha.1` | `1.2.0a1` |
| `1.2.0-beta.1` | `1.2.0b1` |
| `1.2.0-rc.1` | `1.2.0rc1` |

Other pre-release forms and build metadata have no agreed PEP 440 equivalent; do not use
them. Writing the SemVer form into `pyproject.toml` is a mistake that build tools hide
by silently normalizing it.

```json
"profiles": ["python"],
"version_mirrors": [
  {"path": "pyproject.toml", "kind": "pyproject"},
  {"path": "src/<package_name>/__init__.py", "kind": "python_dunder"}
]
```

Drop the second mirror if the package reads its version from metadata with
`importlib.metadata.version`.

## Conventions

- Google-style docstrings on public modules, classes, and functions.
- Type hints on all public functions.
- `pathlib` over `os.path`; `logging` over `print` outside CLIs.

## Gotchas

- `python` may not exist on the machine; use `python3` to create the venv and
  `.venv/bin/python` afterwards.
- `pip install -e .` must be re-run after changing dependencies or entry points.
- Run one test file while iterating; run the whole suite before committing.

## .gitignore

```
.venv/
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.pytest_cache/
.mypy_cache/
.ruff_cache/
.coverage
htmlcov/
```
