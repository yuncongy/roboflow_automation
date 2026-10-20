# Architecture

## Status and design intent

Phase 1 establishes structure and conventions only. The design below guides later phases; there are no working browser fixtures, Page Objects, application locators, API clients, or test scenarios yet. Keep the framework small and extend it only as verified workflows require.

Use Python 3.11+, synchronous Playwright, pytest, pytest-playwright, pytest-xdist, and allure-pytest. See [decisions](DECISIONS.md) for rationale and [roadmap](ROADMAP.md) for implementation gates.

## Directory responsibilities

| Location | Responsibility |
| --- | --- |
| `pages/` | Application Page Objects: verified locators, UI actions, readiness checks, state queries, and meaningful navigation transitions. |
| `components/` | Reusable UI regions shared by pages, with locators scoped to their container. Introduce implementations when actual reuse appears. |
| `tests/` | Pytest scenarios, business assertions, parametrization, and category markers. Tests describe user intent through Page Objects. |
| `fixtures/` | Reusable pytest fixture modules for configuration, Page Objects, disposable resources, and teardown when required. |
| `utils/` | Small application-independent helpers, such as unique names or artifact paths, when a real use arises. |
| `config/` | Future typed configuration and validation. No settings loader or environment contract is implemented in Phase 1. |
| `test_data/` | Small, synthetic, non-sensitive inputs and manifests. No credentials, user datasets, or generated downloads. |
| `reports/` | Ignored generated results, attachments, screenshots, traces, and videos. Retain only `.gitkeep` in Git. |
| `docs/` | Test strategy and chronological progress records. |
| `conftest.py` | Root pytest discovery and future fixture registration/hooks; currently only a module docstring. |
| `pytest.ini` | Minimal collection configuration and registered test categories. |
| `requirements.txt` | Direct dependencies with documented version bounds. |
| Root Markdown files | Entry point, persistent agent rules, architecture, roadmap, and ADRs. |

Python packages contain `__init__.py` only. `test_data/` and `reports/` use `.gitkeep`. No extra source directories or packaging layers are needed now. A local `.venv/`, if created during installation, is an ignored development environment rather than a source layer.

## Boundaries and collaboration

Tests receive dependencies from fixtures, orchestrate Page Objects, and assert outcomes. Page Objects own application UI knowledge and may compose components. Components do not depend on tests or fixtures. Generic utilities must not depend on application Page Objects.

Construct Page Objects with an explicit Playwright `Page`; do not launch browsers or read credentials inside them. A transition may return another Page Object when navigation has a clear destination. Otherwise the test composes the next object using the same injected page. Avoid global registries, navigation managers, circular page imports, and inheritance solely for sharing a few lines.

Keep business assertions in `tests/`. Page Objects may return state or use Playwright expectations to wait for readiness and confirm that a UI action has completed. They must not hide a scenario's acceptance criteria or swallow exceptions. Fixture setup may fail with a clear prerequisite error, but it must not stand in for the scenario's assertions.

## Planned browser and fixture lifecycle

Phase 2 should configure the existing pytest-playwright fixtures rather than create a competing browser manager:

| Dependency | Planned scope | Ownership and teardown |
| --- | --- | --- |
| Playwright runtime and browser | Session, independently within each xdist worker | Plugin starts lazily when requested and closes at worker/session end. |
| Browser context and page | Function | Plugin provides a new isolated context/page per test and closes them afterward. |
| Page Objects and mutable resource fixtures | Function | Wrap that test's page or resources; teardown stays with the owning fixture. |
| Validated immutable settings | Session | Framework fixture validates configuration before application access. |

Prefer context/launch option overrides to replacing plugin lifecycle fixtures. Root `conftest.py` will explicitly register modules from `fixtures/` when they exist; pytest does not discover arbitrary modules there automatically. Imports and test collection must not perform application access or launch browsers.

Use fixture finalizers or `yield` teardown, with cleanup registered immediately after creation so partial setup failures are covered. Any extra contexts or API request contexts must have explicit owners and disposal. Browser isolation does not isolate server-side data; use worker/run/test identifiers for backend resources too.

The planned scopes follow the [Playwright pytest plugin reference](https://playwright.dev/python/docs/test-runners). They are a design decision, not implemented framework behavior.

## API-assisted setup and network validation

Keep API/network outcome assertions in pytest test files. Future fixtures may use verified APIs for setup, teardown, or request-context injection. Put a reusable Roboflow-specific adapter beside its owning fixture module initially; introduce a dedicated API directory only when independent reuse justifies it and an ADR documents the change. Do not hide application API knowledge inside `utils/` or Page Objects.

For a UI-triggered request, the Page Object may synchronize on an observed response and return relevant state; the test decides whether that response satisfies the business requirement. Register listeners before the triggering action, use bounded waits, and redact tokens and payloads from diagnostics. No endpoints or response schemas are assumed in this phase.

## Test data, cleanup, and costly actions

Use synthetic, licensed fixtures in `test_data/`; use pytest temporary paths for generated local files. Version only small stable inputs, documenting provenance and expected use when introduced. Resolve committed data relative to the repository/module, rather than the caller's working directory.

Future resource fixtures must create uniquely named resources in a designated disposable workspace and record exact IDs and ownership. Cleanup is limited to resources created by the current run, including after failure. Never mutate existing user projects, datasets, annotations, models, or organization settings. Authentication state is secret and must be isolated when sessions cannot safely be shared.

Before any costly scenario is implemented, add and validate a default-deny execution gate requiring explicit opt-in and a configured test workspace/budget. Marker registration alone provides categorization, not enforcement. Phase 1 implements neither operations nor gates.

## Introducing abstractions

Start with explicit functions and small classes. Add an abstraction only for demonstrated duplication, a verified integration boundary, or a clear lifecycle responsibility. State its purpose, alternatives, owner, and validation in an ADR for substantial architecture changes. Avoid speculative base pages, service layers, retry wrappers, and generic workflow engines. Prefer transparent Playwright behavior and actionable failures.
