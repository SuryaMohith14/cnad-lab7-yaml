# Automated Workflow Pipelines with YAML: A Practical Guide

This repository is a small, runnable Python project with a GitHub Actions pipeline. It demonstrates how YAML describes **when** a workflow runs, **what** jobs it contains, and **how** each job executes.

## What the pipeline does

On pushes to `main`, pull requests targeting `main`, or a manual run, the workflow:

1. Runs Ruff lint and formatting checks, plus the test suite, on Python 3.11, 3.12, and 3.13.
2. Builds a Python wheel and source distribution only after every validation matrix job succeeds.
3. Uploads both distributions as a downloadable workflow artifact.

The workflow is in `.github/workflows/ci.yml`. Its `needs` dependency prevents publishing a build artifact when validation fails. The workflow has read-only repository permissions, and its concurrency group cancels outdated runs on the same branch or pull request.

## Run the project locally

Requires Python 3.11 or newer.

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e ".[dev]"
ruff check .
ruff format --check .
pytest
python -m temperature_converter.cli c-to-f 25
```

On macOS/Linux, activate the environment with `source .venv/bin/activate`.

Expected CLI output:

```text
25°C = 77°F
```

## Read the YAML from top to bottom

- **`name`** gives the workflow a useful label in the Actions UI.
- **`on`** defines event triggers. `workflow_dispatch` enables a manual run.
- **`permissions`** follows least privilege; this workflow only needs to read repository contents.
- **`concurrency`** groups duplicate runs and stops obsolete ones from consuming runner time.
- **`jobs`** contains independent units of work. GitHub can run the matrix entries concurrently.
- **`strategy.matrix`** repeats the validation job across Python versions, catching compatibility issues.
- **`steps`** are ordered commands or reusable actions. `uses` invokes an action; `run` executes shell commands.
- **`needs: lint-and-test`** makes packaging wait for all lint-and-test matrix entries to pass.
- **`upload-artifact`** stores build outputs with the run so they can be downloaded later.

## Adapt it for your project

1. Replace the package and tests with your application code.
2. Set the Python matrix to versions your project supports.
3. Add required checks (for example, type checking or integration tests) before packaging.
4. Keep secrets out of YAML. Store credentials in repository or environment secrets, and grant write permissions only to a job that needs them.
5. For deployment, add a separate job with `needs` on validation/build, scope it to a protected GitHub environment, and use narrowly scoped permissions.

This example deliberately stops at building and uploading an artifact: deployment credentials and target environments vary by project.
