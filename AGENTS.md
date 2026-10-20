# Engineering instructions

These rules apply to all work in this repository and all future agent sessions.

## Scope and working process

- Read this file, `docs/PROGRESS.md`, `ARCHITECTURE.md`, and the relevant roadmap phase before making changes.
- Complete only the phase or scope authorized by the user. Phase 1 is foundation only; do not implement browser automation, Page Object methods, application locators, fixtures, or test scenarios without a separate instruction.
- Every substantial change must have a clear purpose, be validated with appropriate checks, and be summarized before proceeding to the next substantial change. Record outcomes and limitations in `docs/PROGRESS.md`.
- Prioritize readability, maintainability, and test reliability over implementation speed.
- Maintain documentation as the framework evolves. Record meaningful architecture changes in `DECISIONS.md`.

## Design and implementation

- Follow Page Object Model (POM). Keep browser locators and UI actions inside Page Objects or reusable components.
- Keep test assertions and business acceptance decisions in pytest test files. Page Objects may expose state-query methods and use Playwright expectations to establish readiness or action postconditions.
- Use pytest fixtures for setup, teardown, and dependency injection. Prefer pytest-playwright's existing fixtures over a custom browser lifecycle.
- Prefer synchronous Playwright APIs initially. Use explicit dependencies and type annotations when they improve clarity.
- Use semantic locators (`get_by_role`, `get_by_label`, `get_by_test_id`) when possible, verified against the actual application.
- Do not invent Roboflow selectors, APIs, or behavior. Verify them against authorized observation or official documentation before implementation.
- Avoid fixed sleeps, fragile selectors, hardcoded credentials, unnecessary abstractions, and hidden global state. Use Playwright auto-waiting and bounded expectations for synchronization.
- Keep application-specific logic in pages, components, or the fixtures that own it; keep generic utilities independent of Roboflow.
- Do not introduce directories, base classes, managers, or architectural layers without a concrete need and a documented reason.

## Isolation and safety

- Keep tests independent, repeatable, and safe for parallel execution. Use a fresh browser context per test; never share mutable Page Objects, resource names, or output files across workers.
- Never modify or delete existing user data. Future writes must use explicitly designated disposable test resources in an authorized workspace.
- Require explicit opt-in before costly cloud operations, including training, paid inference, deployments, or quota-consuming jobs. A marker alone is not an opt-in gate. Implement and validate enforcement before enabling such tests.
- Register cleanup as soon as a test resource is created. Delete only exact resource IDs owned by the current run; do not clean up by broad name matching. Report cleanup failures without hiding the original failure.
- Keep credentials, tokens, cookies, authentication state, and private datasets out of source control and logs. Use environment variables or an approved secret provider when configuration is implemented.
- Treat traces, screenshots, videos, and Allure attachments as potentially sensitive. Store generated artifacts in ignored locations and review them before sharing.

## Validation and handoff

- Run checks appropriate to the change; never claim a check passed unless it ran. Distinguish dependency resolution, collection, and executed tests.
- No tests exist in Phase 1. Pytest's no-tests exit code is expected, not a passing test suite. Do not add placeholder tests or suppress this exit code to manufacture success.
- Summarize what changed, why, validation evidence, limitations, and next actions. Update progress and decisions before the final handoff.
