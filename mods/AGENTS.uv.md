# Instructions for development with Python and uv

The Python environment is managed with `uv`.

There is no `pip install`, no `python -m venv`, no `conda`.

## Dependencies

The following development dependencies should be included:

- `pytest`
- `deptry`

## Common commands

Use `uv` everywhere:

- Sync the environment from: `uv sync`
- Add a dependency: `uv add <pkg>`
- Run a CLI tool: `uv run <cmd>`
- Run a Python script: `uv run python <script>.py`
- Run a Python module: `uv run -m <mod>`

## Tests and dependency checks

- Run tests: `uv run pytest`.
- Check dependencies: `uv run deptry .`
