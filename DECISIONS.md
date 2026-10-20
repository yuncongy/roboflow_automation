# Architecture decisions

## Record format

Use sequential IDs (`ADR-001`, etc.). Keep accepted records as history; add a superseding record when a decision changes.

```text
## ADR-NNN: Short title
Date: YYYY-MM-DD
Status: Proposed | Accepted | Superseded by ADR-NNN
Context: The concrete problem or constraint.
Decision: The choice and its scope.
Alternatives: Meaningful options considered.
Consequences: Benefits, tradeoffs, and follow-up obligations.
Validation: Evidence obtained or checks required before implementation.
```

## ADR-001: Python and the pytest/Playwright stack

- **Date:** 2026-10-19
- **Status:** Accepted
- **Context:** The project needs extensible UI automation with fixtures, parallel execution, and reporting using the requested technology stack.
- **Decision:** Use Python 3.11+, Playwright for Python, pytest, pytest-playwright, pytest-xdist, and allure-pytest. Use Git for source control when initialized. Use synchronous Playwright APIs initially.
- **Alternatives:** Async Playwright adds event-loop management without a current requirement. A custom runner or browser lifecycle would duplicate existing plugins.
- **Consequences:** Browser binaries are a separate installation. allure-pytest produces result data; rendering reports requires a separately installed Allure report tool. Dependency installation alone does not validate UI execution.
- **Validation:** Python 3.11.9 is available. Consult package metadata and perform dependency resolution where permitted; record exact evidence in `docs/PROGRESS.md`. Browser/plugin runtime integration is deferred to Phase 2.

## ADR-002: Flat packages and Page Object boundaries

- **Date:** 2026-10-19
- **Status:** Accepted
- **Context:** The initial scope needs clear ownership without speculative infrastructure.
- **Decision:** Use only the requested flat directories. Tests own scenarios and business assertions; pages/components own locators and UI actions; fixtures own dependency injection and lifecycle; utilities remain application-independent. Page Objects may query state and use Playwright expectations for readiness/postconditions.
- **Alternatives:** A `src/` layout, base-page hierarchy, workflow engine, or separate API layer could become useful later but has no demonstrated need now.
- **Consequences:** Run commands from the repository root via `python -m pytest`. Fixture modules will need explicit registration when implemented. A significant new layer requires documented justification.
- **Validation:** Phase 1 checks the exact skeleton and absence of implementation. Validate boundaries against the first real slice in Phase 3.

## ADR-003: Plugin-owned lifecycle and per-test isolation

- **Date:** 2026-10-19
- **Status:** Accepted
- **Context:** Parallel browser tests need clear ownership and isolation without duplicating Playwright's fixture system.
- **Decision:** Plan to reuse pytest-playwright's session browser/runtime per worker and function-scoped contexts/pages. Construct Page Objects through function-scoped fixtures. Keep backend resources and artifact paths unique per run, worker, and test.
- **Alternatives:** A browser per test costs more; a shared context risks state leakage. A global singleton obscures lifecycle and breaks worker isolation.
- **Consequences:** Session scope is per xdist worker, not global across the run. Browser isolation alone cannot prevent backend resource collisions.
- **Validation:** Scopes match the [official plugin reference](https://playwright.dev/python/docs/test-runners). Validate actual lifecycle, teardown, and worker output separation in Phase 2. No custom fixtures exist yet.

## ADR-004: Requirements file with bounded direct dependencies

- **Date:** 2026-10-19
- **Status:** Accepted
- **Context:** Initial setup must use standard pip requirements without introducing a packaging or lock-tool layer.
- **Decision:** Declare the five direct dependencies in `requirements.txt` with lower bounds and upper major-version bounds; cap the pre-1.0 pytest-playwright plugin below 1.0. Allow pytest 8.4 through 9.x. Resolve them together for the available Python/platform.
- **Alternatives:** Unbounded dependencies invite uncontrolled major changes. Full transitive pins improve reproducibility but require a reviewed update and platform policy. A new package manager is outside current scope.
- **Consequences:** This file is not a lockfile. Different installation dates may resolve different versions, and accepted ranges do not prove every combination works. Record the validated resolution and revisit a reproducible snapshot before CI adoption.
- **Validation:** Check PEP 508 syntax and resolver compatibility; record selected versions and limitations in the progress log. Runtime and supported-Python matrix checks remain future work.

## ADR-005: Disposable data and default-deny costly workflows

- **Date:** 2026-10-19
- **Status:** Accepted
- **Context:** UI workflows can alter valuable user data or consume cloud credits.
- **Decision:** Use only authorized disposable test resources; never alter existing user data. Future fixtures track exact created IDs and clean up only owned resources. Require explicit opt-in before costly cloud operations and validate enforcement before enabling them. Keep secrets and generated artifacts out of Git.
- **Alternatives:** Reusing existing projects or inferring consent from a pytest marker would leave ownership and spending ambiguous.
- **Consequences:** A dedicated workspace, secret contract, cleanup behavior, and budget controls must be agreed before application-changing scenarios. The `costly` marker is categorization only; enforcement is planned for Phase 2.
- **Validation:** Phase 1 reviews policy consistency and ignore rules. Phase 2 must validate default denial, and later workflow phases must exercise cleanup under failure.
