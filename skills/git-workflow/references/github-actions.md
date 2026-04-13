# GitHub Actions CI

Reference for the CI pipeline. Consult when debugging CI failures or modifying workflow.

## Pipeline: `ci.yml`

Three sequential jobs: **lint → test → build**

### lint
- Python 3.14 (prereleases allowed)
- `ruff check src/ tests/` + `ruff format --check src/ tests/`

### test
- Depends on lint passing
- Installs `pip install -e ".[embeddings]" --group dev`
- `pytest tests/ -q --tb=short` with `PHILEAS_TEST_MODE=1`

### build
- Depends on test passing
- `python -m build`
- Uploads `dist/` as artifact

## Concurrency

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

New pushes to the same branch cancel in-flight runs.

## Common failure patterns

| Symptom | Cause | Fix |
|---------|-------|-----|
| `ruff check` fails | Lint violations (E501, F401, etc.) | `ruff check --fix src/ tests/` |
| `ruff format` fails | Formatting drift | `ruff format src/ tests/` |
| Test fails only in CI | Test reads `~/.phileas/config.toml` instead of using `tmp_path` | Pass `home=tmp_path` to `load_config()` |
| Build fails | Missing `pyproject.toml` metadata | Check `[project]` table |

## Useful commands

```bash
# Check CI status for a PR
gh pr checks <number>

# View failed job logs
gh run view <run-id> --log-failed

# Search logs for errors
gh run view <run-id> --log 2>&1 | grep -E "FAILED|Error"

# Re-run failed jobs
gh run rerun <run-id> --failed
```
