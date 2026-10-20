# Test strategy

## Status

This document defines future coverage. Phase 1 contains no test cases, application locators, Page Object implementations, browser fixtures, API adapters, or visual comparison tooling. Marker registration is implemented; behavior and enforcement are planned.

## Categories

| Category / marker | Purpose | Expected evidence and boundaries |
| --- | --- | --- |
| Smoke / `smoke` | Small, fast set confirming agreed critical paths in a disposable workspace. | Visible outcome assertions; bounded execution; no costly operations by default. Establish the actual paths after UI verification. |
| Functional E2E / `e2e` | Complete agreed user workflows through Page Objects and real UI interactions. | Business assertions in tests; deterministic prerequisites; owned resources and failure-safe cleanup. A smoke test may also be E2E. |
| API-assisted / `api_assisted` | Use verified APIs for setup/teardown or to validate UI-triggered backend outcomes. | Outcome assertions remain in tests; setup cannot replace the UI behavior being tested. Only documented or observed authorized endpoints and contracts are allowed. |
| Visual regression / `visual` | Detect unintended visual changes on selected stable regions. | Reviewed baselines, fixed browser/OS/fonts/viewport, deterministic data, intentional masking of volatile regions, and explicit diff thresholds. Tooling and baselines are deferred to Phase 7. |

Apply `costly` in addition to a category for training, paid inference, deployment, or other quota/cost-sensitive operations. It does not itself prevent execution. A default-deny gate and explicit opt-in must exist before any costly scenario is introduced. No such scenarios or gate exist in Phase 1.

## Test construction

Use descriptive `test_*.py` files and `test_*` functions. Tests state user intent and acceptance criteria; Page Objects/components contain browser actions and verified locators. Use fixtures for all prerequisite setup, teardown, and injected dependencies. Favor semantic locators and bounded Playwright expectations; avoid sleeps and retries that conceal defects.

Keep real-service E2E coverage distinct from mocked infrastructure checks. Assert outcomes, including relevant negative paths, rather than implementation details. Record the scope and evidence when a workflow becomes supported. No claims about Roboflow functionality are inferred from a successful local infrastructure check.

## Isolation and parallel execution

- Use a fresh browser context/page for every test and no dependencies on test ordering.
- Namespace backend resources with run, worker, and test identifiers; retain exact returned IDs.
- Do not mutate shared accounts, projects, datasets, annotations, or organization settings. Allocate separate test identities where concurrent login/session behavior requires it.
- Keep output paths unique across workers and browser variants. Use pytest temporary directories for transient local data.
- Make timeouts and data deterministic. Account for service-side eventual consistency with bounded checks only after verifying the actual behavior.
- Introduce worker concurrency gradually and validate quota constraints. Installing xdist is not evidence that scenarios are parallel-safe.

## Setup and cleanup

Use small synthetic inputs whose provenance is documented. Create resources only in an authorized disposable workspace; existing user data is never a test fixture to mutate. Register cleanup immediately after each successful creation and preserve exact ownership information through partial failures.

Attempt cleanup on both pass and failure, using verified UI or API behavior. Never delete by a broad prefix or assume names establish ownership. Report cleanup errors and owned residual IDs while preserving the primary test failure. Establish a targeted recovery procedure before adding workflows that can leave jobs or resources behind.

Credentials, cookies, storage state, and private data must be supplied through the eventual approved secret/configuration contract. Do not commit them. Traces, screenshots, logs, and attachments may contain sensitive data; keep them ignored locally and define access/retention before CI publishing.

## Validation and reporting

Phase 1 validates the skeleton, Python/configuration syntax, documentation consistency, requirements resolution where possible, and ignore behavior. `--collect-only` can verify pytest configuration without running scenarios; when there are no tests, exit code 5 is expected, as described in the [pytest exit-code reference](https://docs.pytest.org/en/stable/reference/exit-codes.html). Never convert this into a successful test result.

Phase 2 will validate infrastructure on a controlled local target, including teardown, two-worker isolation, configuration errors, and default-deny cost controls. Later phases validate agreed application workflows with an authorized test account. Log actual commands, outcomes, environment, and limitations in [PROGRESS.md](PROGRESS.md).

Future Allure results and Playwright diagnostics belong under `reports/` with unique run paths. Allure integration and CI publication are planned; the Python dependency alone does not provide rendered reports. Failures must remain visible; any future retry policy requires evidence, a documented limit, and reporting of the original failure.
