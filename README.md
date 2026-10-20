# roboflow_automation

A Python UI automation framework for the Roboflow computer vision web application, inspired by an enterprise testing framework for ArcGIS Deep Learning Studio. The goal is reliable, maintainable automation that grows through verified workflows.

**Status: Phase 1 — project foundation only.** Implemented: directory/package skeleton, engineering rules, design records, dependency declarations, and minimal pytest configuration. Browser automation, Page Object methods, application locators, fixtures, tests, API helpers, cost controls, visual comparison, and CI/report generation are planned. No tests exist yet.

## Technology stack

Python 3.11+, Playwright for Python (synchronous API), pytest, pytest-playwright, pytest-xdist for parallel execution, allure-pytest for reporting integration, Page Object Model, and Git. Direct dependencies use standard [requirements.txt](requirements.txt); see [ADR-004](DECISIONS.md#adr-004-requirements-file-with-bounded-direct-dependencies) for version bounds and reproducibility limits.

## Folder layout

```text
roboflow_automation/
├── pages/
│   └── __init__.py
├── components/
│   └── __init__.py
├── tests/
│   └── __init__.py
├── fixtures/
│   └── __init__.py
├── utils/
│   └── __init__.py
├── config/
│   └── __init__.py
├── test_data/
│   └── .gitkeep
├── reports/
│   └── .gitkeep
├── docs/
│   ├── TEST_STRATEGY.md
│   └── PROGRESS.md
├── AGENTS.md
├── ARCHITECTURE.md
├── ROADMAP.md
├── DECISIONS.md
├── conftest.py
├── pytest.ini
├── requirements.txt
├── .gitignore
└── README.md
```

See [ARCHITECTURE.md](ARCHITECTURE.md) for directory ownership, fixture lifecycle, Page Object collaboration, API boundaries, data management, and criteria for new abstractions.

## Local environment installation

Run from `/Users/yuncongyu/Projects/roboflow_automation` using Python 3.11 or newer. The local `.venv/` is ignored and is not part of the source architecture.

```bash
cd /Users/yuncongyu/Projects/roboflow_automation
python3 --version
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip check
```

On Windows, activate with `.venv\Scripts\Activate.ps1` in PowerShell and adjust the repository path. Dependency declarations do not enforce the project-wide Python minimum, so verify the interpreter before installation.

Browser binaries are separate from Python packages. Install them when Phase 2 is authorized and browser execution is needed:

```bash
python -m playwright install chromium
```

The supported host and any OS dependencies must be checked during Phase 2. Do not download browsers merely to validate this foundation. Installation follows the [official Playwright Python guide](https://playwright.dev/python/docs/intro).

The Allure Python adapter writes result data. Rendering/viewing reports requires a separately installed report tool; follow the [official Allure pytest instructions](https://allurereport.org/docs/pytest/) when reporting is implemented.

The project has a local `.venv/` using Python **3.11.9**, with all dependencies from `requirements.txt` installed. `pip check` reports no broken requirements, and pytest collection loads the configuration/plugins successfully with zero tests (exit code 5). Browser binaries are not installed, and browser execution has not been validated.

Git is initialized on `main`, tracking `origin/main` at [yuncongy/roboflow_automation](https://github.com/yuncongy/roboflow_automation). The existing initial commit is preserved as the base for the foundation files. `.venv/`, caches, credentials, and generated reports are ignored.

For an existing local setup, activate the environment in each new terminal:

```bash
cd /Users/yuncongyu/Projects/roboflow_automation
source .venv/bin/activate
python --version
python -m pytest --collect-only -q
```

## Validation now and test execution later

After installing dependencies, this checks discovery and configuration without running tests or launching browsers:

```bash
python -m pytest --collect-only -q
```

At Phase 1 it should collect **zero tests** and exit with **code 5**. This is pytest's [documented no-tests result](https://docs.pytest.org/en/stable/reference/exit-codes.html), not a passing test suite. `conftest.py` contains no fixtures or hooks, and no placeholder tests are included.

After infrastructure and actual scenarios are separately authorized and implemented, the intended command forms are:

```bash
python -m pytest tests/ -m smoke --browser chromium
python -m pytest tests/ -m "e2e and not costly" -n 2 --browser chromium
python -m pytest tests/ -m "smoke and not costly" --browser chromium --alluredir reports/allure-results --output reports/playwright --tracing retain-on-failure --screenshot only-on-failure
```

These illustrate future execution; they do not demonstrate a working suite. Run-specific artifact paths and infrastructure policy will be implemented in Phase 2. Registered markers are `smoke`, `e2e`, `api_assisted`, `visual`, and `costly`; category definitions are in [TEST_STRATEGY.md](docs/TEST_STRATEGY.md). A marker is not a cost gate. No costly workflow may be introduced before default-deny enforcement and explicit opt-in are validated.

No environment-variable names, application URLs, credentials, selectors, or API endpoints are prescribed yet. Phase 2 will define and validate configuration; application phases require verified behavior and an authorized disposable workspace. Existing user data must never be modified.

## Roadmap and agent conventions

The [roadmap](ROADMAP.md) defines objectives, deliverables, dependencies, and acceptance criteria for eight phases: foundation; core infrastructure; Page Objects; authentication/project lifecycle; annotation/datasets; training/inference; advanced capabilities; CI/CD/reporting. Stop after the authorized phase.

Read [AGENTS.md](AGENTS.md) before making changes. Keep UI actions in pages/components, business assertions in pytest tests, lifecycle in fixtures, and generic helpers in utilities. Prefer semantic locators and bounded expectations. Every substantial change needs a clear purpose, validation, and a summary before proceeding. Keep [PROGRESS.md](docs/PROGRESS.md) current and record meaningful architecture choices in [DECISIONS.md](DECISIONS.md).

Phase 2 should configure plugin-owned fixtures, validated settings/secrets, reliable teardown, unique worker artifact paths, and explicit cost opt-in enforcement, then validate them on a controlled local target. It has not started.
