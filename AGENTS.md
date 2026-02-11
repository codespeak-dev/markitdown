A virtual environment is pre-configured at the project root (`.venv/`). Hatch is installed there.

# Running Tests

From `packages/markitdown/`, run `GITHUB_ACTIONS=1 hatch test`. Skipping remote URL testing is necessary for any new work.

The full test suite takes several minutes, but it's important and expected from all collaborators to run ALL of the tests.
As such, agents MUST wait until the finish instead of interrupting the process and running some selection of tests.

# Adding Tests

The primary testing mechanism is the **test vector framework**:

1. Add test fixture files to `tests/test_files/`
2. Add `FileTestVector` entries to `tests/_test_vectors.py`

The parametrized tests in `test_module_vectors.py` will automatically exercise your converter through all standard code paths.
