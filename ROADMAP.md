# Development roadmap

Phases are sequential scope gates, not permission to implement future work. Phase 1 is the only currently authorized phase. Each later phase requires a separate instruction; verify Roboflow behavior before implementing application features. Acceptance criteria are minimum exit conditions, and workflow scope must be agreed before implementation.

## 1. Project foundation — current phase

- **Objective:** Establish a maintainable, documented foundation without automation behavior.
- **Deliverables:** Requested directory skeleton, persistent instructions, architecture, ADRs, roadmap, progress log, test strategy, dependency declarations, pytest configuration, and Git ignore rules.
- **Dependencies:** Existing project directory, Python 3.11+, and available package metadata for compatibility checks.
- **Acceptance:** All required paths exist; Python stubs and configuration parse; documentation links and phase boundaries agree; dependency compatibility is checked to the extent possible and limitations recorded; no browser fixtures, locators, or test scenarios are implemented. Git initialization status is recorded separately if permissions prevent it.

## 2. Core Playwright and pytest infrastructure — planned

- **Objective:** Make configuration, isolated browser lifecycle, and diagnostics dependable.
- **Deliverables:** Validated settings and secret-input contract; configuration of pytest-playwright fixtures; fixture registration; deterministic artifact paths; default-deny costly-operation gate; documented browser installation and execution instructions.
- **Dependencies:** Phase 1 accepted, separate authorization, supported browser host, and an agreed safe configuration contract. Local infrastructure validation should not require Roboflow credentials.
- **Acceptance:** A local controlled infrastructure check launches and closes a browser; two test invocations receive different contexts; two workers use distinct output paths; missing/invalid settings fail clearly; costly operations are denied without explicit opt-in. No Roboflow workflow is needed for this phase.

## 3. Page Object architecture — planned

- **Objective:** Establish small, verified application objects through an agreed initial navigation slice.
- **Deliverables:** First Page Object and any demonstrated reusable component; typed state-query/action conventions; fixtures constructing these objects; locator verification evidence.
- **Dependencies:** Phase 2, authorized application access, and observed UI behavior for the selected slice.
- **Acceptance:** The selected slice runs through Page Objects; its UI locators/actions are absent from test files; business assertions remain visible in tests; no speculative base classes or unverified selectors are introduced.

## 4. Authentication and project lifecycle testing — planned

- **Objective:** Validate authorized sign-in and disposable project lifecycle behavior.
- **Deliverables:** Verified authentication strategy, secret/state handling, project creation/query/deletion scenarios, and resource ownership/cleanup fixtures.
- **Dependencies:** Phase 3; dedicated test account/workspace; confirmed authentication and account constraints, including any SSO/MFA limits.
- **Acceptance:** Agreed authentication and project scenarios pass on three consecutive runs; setup failure and test failure both exercise cleanup; two workers do not share mutable resources; only run-owned projects are changed or deleted.

## 5. Annotation and dataset workflows — planned

- **Objective:** Cover agreed image import, annotation, and dataset preparation workflows.
- **Deliverables:** Verified pages/components, small synthetic data with provenance, category-marked scenarios, and cleanup for all created resources.
- **Dependencies:** Phase 4, agreed annotation/task types, verified UI/API behavior, and approved non-sensitive input data.
- **Acceptance:** Each agreed workflow has a visible outcome assertion; three repeated runs and a two-worker run do not contaminate each other; failure-path cleanup removes only run-owned resources; dataset sizes remain within agreed limits.

## 6. Training and inference workflows — planned

- **Objective:** Validate an agreed minimal training/inference flow under explicit cost controls.
- **Deliverables:** Verified workflow objects, bounded job polling, explicit opt-in/budget checks, useful failure diagnostics, and cleanup/cancellation rules where supported.
- **Dependencies:** Phase 5; explicit authorization for each costly run; confirmed quotas, cost limits, job behavior, and allowed models/data.
- **Acceptance:** Without opt-in, no job or paid request starts; an authorized minimal run verifies agreed output; timeout/cancellation paths are validated using controlled checks before cloud execution; run IDs, limits, and any unavoidable residual resources are reported.

## 7. Advanced framework features — planned

- **Objective:** Add capabilities supported by measured needs and observed failures.
- **Deliverables:** Selected API-assisted validation, visual baselines/comparison tooling, cross-browser coverage, and reliability improvements with supporting ADRs.
- **Dependencies:** Stable earlier workflows, a prioritized feature list, verified API contracts, and an agreed baseline platform for visual checks.
- **Acceptance:** Each selected feature has a measurable success criterion; visual checks demonstrate both a known pass and an intentional mismatch; API checks validate observed contracts; reliability changes report before/after evidence without hiding failures behind retries.

## 8. CI/CD and reporting — planned

- **Objective:** Run a controlled suite reproducibly and publish actionable results.
- **Deliverables:** CI configuration, secret injection, compatible browser setup, reviewed dependency snapshot/locking policy, Allure report generation, artifact retention/redaction, and documented run procedures.
- **Dependencies:** Stable smoke suite, chosen CI platform and runner matrix, approved test workspace/secrets, and agreed reporting/retention policies.
- **Acceptance:** A clean runner installs dependencies and browsers and executes the selected suite; an intentional failure produces a failing job and inspectable report; artifacts are access-controlled and expire on schedule; costly jobs remain opt-in; supported Python/browser combinations are recorded.
