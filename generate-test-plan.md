---
name: generate-test-plan
description: Create or update a test plan from requirements documents, design documents, and one or more local code repositories. Analyze implementation and existing test coverage.
---

# Generate Test Plan

## 1. Purpose

Create a test plan grounded in the supplied project materials:

- **Requirements** describe expected product behavior and constraints.
- **Design documents** describe intended components, interfaces, and flows.
- **Source code and configuration** show the current implementation.
- **Existing tests** show which behaviors are currently asserted.

Connect these sources where possible:

`Requirement → Design → Component / Interface → Code → Existing Test → Planned Test`

## 2. Inputs and scope

### 2.1 Inputs

Use any combination of:

- Requirements documents.
- Design documents, architecture diagrams, API specifications, schemas, sequence diagrams, or ADRs.
- One or more local repository paths.
- An optional feature name, scope, baseline commit, output path, or team template.

Inspect a document format only when it is accessible to the current agent and tools. If a file cannot be read or a diagram cannot be inspected, record that limitation.

Example input patterns (adapt syntax to the host agent):

```text
requirements: docs/requirements/task-routing.md
design: docs/design/system-architecture.md
repositories: ./gateway ./scheduler ./runtime
```

### 2.2 Resolve scope

1. Start with the paths and scope supplied by the user.
2. If a repository is supplied without document paths, look in a limited set of likely locations: `docs/`, `design/`, `spec/`, `specs/`, `requirements/`, `architecture/`, `adr/`, `api/`, and the repository root.
3. Do not scan dependency, cache, build, or generated directories unless they are relevant: `.git/`, `node_modules/`, `.venv/`, `venv/`, `vendor/`, `dist/`, `build/`, and `target/` are typical exclusions.
4. If no repository path is supplied, use the current workspace only after confirming its root. Do not scan a parent directory or home directory by default.
5. Record the documents, repositories, and directories actually inspected. Do not imply that uninspected inputs were reviewed.
6. If information is incomplete but the feature can be identified, produce a preliminary plan and list the gaps. Ask a question only when the scope cannot be established.

### 2.3 Repository baseline

- Use the branch or commit specified by the user; otherwise use remote `main` (prefer `origin/main`).
- Fetch the selected `main` branch when needed and permitted. If no suitable remote `main` is available, report the limitation rather than silently using the local branch.
- Record the remote, branch, and commit SHA. Report uncommitted local changes separately and exclude them unless the user asks to include them.
- Do not switch branches or modify the working tree.

## 3. Evidence rules

### 3.1 Use each source appropriately

| Source | Use it to establish | Do not use it to establish |
|---|---|---|
| Requirements | Expected behavior, constraints, acceptance conditions | Implementation details not stated in the requirements |
| Design | Intended components, interfaces, data flows, and failure behavior | A resolution to a conflicting requirement without noting the conflict |
| Code and configuration | Implemented behavior, dependencies, schemas, and runtime configuration | Product intent when requirements are absent |
| Existing tests | Behaviors asserted by the test code | Successful execution unless test results are available |

When sources disagree, describe the disagreement and cite both locations. Do not silently choose one source. Treat requirements as intended behavior and code as current implementation.

### 3.2 Separate facts, observations, and recommendations

Use these labels where the distinction matters:

- **Documented:** stated in a requirement or design source.
- **Observed:** found in source code, configuration, or an existing test.
- **Derived:** a test condition logically derived from documented or observed behavior.
- **Recommended:** a risk-based test suggestion that is not explicitly required by the source.
- **Unknown:** cannot be established from the inspected material.

Do not present a recommendation or assumption as an approved requirement. Do not invent API paths, payload fields, status codes, commands, environment variables, topology, performance targets, retry behavior, or supported platforms. Use `TBD` when a value is not available and state what needs confirmation.

Cite evidence with repository-relative paths and, where practical, headings, symbols, line ranges, schema fields, or test names. For PDFs and Office documents, include page or section references when available.

### 3.3 Assess document quality

For each requirements or design document, record its title and version/status/date only when stated. Prefer an explicitly approved source over an unapproved draft; a newer file timestamp does not by itself make a document authoritative.

Identify relevant placeholders, unresolved decisions, missing acceptance conditions, and contradictions. If a document is mostly empty, create a preliminary plan only for the sections with usable content.

## 4. Analysis workflow

Follow these steps in order. Scale the depth of analysis to the feature and the supplied material.

### Step 1 — Understand the system and scope

Summarize the feature objective, users or external systems, in-scope components, dependencies, and exclusions using available evidence.

Identify the main components and relevant data/control flows. Use a Mermaid diagram only when it clarifies the design. Do not add components or connections that cannot be supported by the inspected sources.

### Step 2 — Extract requirements

Preserve requirement IDs from the source. If none exist, create stable IDs such as `REQ-001`, `REQ-002`. Keep IDs stable when updating an existing plan.

For each requirement, record:

- ID and concise statement.
- Source document and section/page.
- Requirement type, if useful (functional, API, data, security, performance, reliability, operations, compatibility).
- Acceptance condition or observable result.
- Related design elements and components, where identified.
- Testability: `Testable`, `Ambiguous`, `Not Measurable`, or `Not Testable`.

Do not add numeric limits to terms such as “high performance”, “high availability”, or “secure” unless a source defines them. List missing thresholds or acceptance conditions as open questions.

Statements about engineering work (for example, “add unit tests”) are quality activities, not product behavior. Track them separately where relevant.

### Step 3 — Review design

For each in-scope requirement, identify related components, interfaces, schemas, state transitions, data flows, and external dependencies.

Review applicable details such as input validation, authentication and authorization, error handling, timeouts, retries, idempotency, concurrency, persistence, recovery, observability, and compatibility when they appear in the design.

Identify failure behavior described by the design. When expected behavior is not specified, record the gap. A possible failure test may still be listed as a **Recommended** scenario, with the expected behavior marked `TBD` until confirmed.

### Step 4 — Analyze repositories

Inspect the relevant parts of each repository rather than attempting to read every file.

Identify, where applicable:

- Languages, frameworks, application entry points, and service boundaries.
- APIs, schemas, protocol definitions, generated clients, and message formats.
- Configuration, deployment manifests, container files, and infrastructure definitions.
- Data models, persistence, queues, caches, and external adapters.
- Test directories, fixtures, mocks, and test utilities.

For each requirement or design element, locate the relevant implementation where possible. Record repository, path, and symbol or line reference. Use one of these statuses:

- `Located` — implementation location identified.
- `Partially Located` — only part of the flow or behavior identified.
- `Not Located` — no implementation location found in the inspected scope.
- `Cannot Verify` — insufficient access or context to assess.

A missing code location does not prove that the feature is unimplemented. It may reside in another repository, generated code, or an external system.

### Step 5 — Review existing tests

Search relevant repositories for unit, API/contract, integration, end-to-end/system, performance, reliability, security, and compatibility tests as applicable.

Inspect test setup and assertions. A filename or test name alone is not sufficient evidence of coverage.

For each proposed scenario, classify existing test coverage as:

- `COVERED` — an existing test asserts the same behavior and expected result.
- `PARTIAL` — a related test exists but misses a relevant condition or assertion.
- `GAP` — no matching test was found in the inspected scope.
- `UNKNOWN` — test discovery or inspection was incomplete.

Record the test path and test name, and explain partial coverage. Do not claim that a test passes unless its execution result is available. Do not run tests unless the user asks for test execution.

Find test commands from repository documentation, build files, Makefiles, package scripts, or task runners. Do not guess commands from the language or framework.

### Step 6 — Build traceability

Create a matrix linking:

`Requirement → Design reference → Component → Repository/code location → Existing test → Planned test`

Rules:

1. Map every testable in-scope requirement to at least one scenario, or explain why no scenario is proposed.
2. A scenario may cover multiple requirements; list each one.
3. A requirement may involve multiple repositories and test levels; keep ownership and boundaries clear.
4. Every scenario must map to a requirement, documented design risk, observed behavior requiring regression coverage, or a clearly labeled recommendation.
5. Do not treat implementation presence as test coverage.
6. Keep requirement coverage, existing test coverage, and test execution results separate.

Report counts only when the inspected scope and denominator are clear. Do not imply complete repository coverage after a partial scan.

### Step 7 — Design and prioritize scenarios

Consider applicable success paths, invalid inputs, boundaries, state transitions, concurrency, duplicate requests, dependency failures, recovery, data integrity, compatibility, and non-functional behavior. Select cases based on the requirements and design; do not add every category by default.

Use the lowest-cost test level that can validate the behavior reliably:

| Level | Typical use |
|---|---|
| Static checks | Formatting, types, lint, and static analysis |
| Unit | Isolated logic, validation, transformations, and state transitions |
| API / contract | Interface behavior, schemas, errors, and compatibility |
| Component / integration | Interactions among modules or services and controlled dependencies |
| System / E2E | Cross-component workflows that require the composed system |
| Performance / reliability / security | Non-functional requirements and defined failure models |
| Compatibility / upgrade | Version, configuration, data, or API compatibility when in scope |

Assign a priority to each scenario:

| Priority | Meaning |
|---|---|
| P0 — Critical | Failure has a critical effect on core operation, security, or data integrity. |
| P1 — High | Failure affects a major supported workflow or important integration. |
| P2 — Medium | Failure has limited impact or a reasonable workaround. |
| P3 — Low | Failure has low impact or affects a non-critical edge case. |

Add a brief rationale for P0/P1. Mark priorities as provisional when impact or likelihood is unclear.


## 5. Project-specific test dimensions

Use this section as a set of prompts. Apply a dimension only when the requirements, design, or code show that it is relevant.

### 5.1 APIs and service interfaces

Consider valid and invalid inputs, schema validation, authorization, error contracts, idempotency, concurrency, timeout/retry behavior, pagination, and compatibility where applicable.

### 5.2 Distributed services and task routing

When services route tasks across components or execution locations, consider routing constraints, task identity, status/result association, duplicate submission, failure handling, and recovery. Do not assume transparent routing, automatic failover, or zero application changes unless specified.

### 5.3 Space/ground and intermittent connectivity

When satellite/ground links or intermittent networks are in scope, consider latency, packet loss, disconnection, partition during task execution or data transfer, delayed or duplicated messages, reconnection, state reconciliation, and data integrity. Expected recovery behavior must come from the design or be marked as requiring confirmation.

### 5.4 Simulation modes

When multiple simulation modes exist, document what each mode represents and which components, interfaces, timing, and failure behaviors it exercises. Do not treat one mode as evidence for another without support.

### 5.5 Compute resources and scheduling

When compute allocation or scheduling is in scope, consider resource discovery, eligibility, placement constraints, allocation/release, capacity, contention, workload lifecycle, and recovery. Include accelerators, quotas, priorities, or multi-tenancy only when they exist in the project scope.

## 6. Test scenario format

### 6.1 IDs

Preserve IDs from an existing plan. For a new plan, use stable IDs such as `TC-001`, `TC-002`. Do not renumber unchanged scenarios when updating a plan. Mark removed or superseded scenarios with a reason rather than reusing their IDs.

### 6.2 Required fields

Each scenario should include:

| Field | Description |
|---|---|
| ID and title | Stable ID and behavior-focused title |
| Priority | P0–P3 and rationale for P0/P1 |
| Type / level | For example, negative / API or resilience / integration |
| Traceability | Requirement IDs and relevant design references |
| Component / repository | Owning component and repository |
| Target test path | Existing path or a proposed path based on repository conventions; otherwise `TBD` |
| Existing coverage | `COVERED`, `PARTIAL`, `GAP`, or `UNKNOWN`, with evidence |
| Implementation status | `EXISTS`, `TO ADD`, `MANUAL`, `BLOCKED`, or `TBD` |
| Setup and test data | Required environment, state, configuration, and input data |
| Steps and expected results | Actions and observable assertions |
| Cleanup / dependencies | Required cleanup and relevant blockers |

Keep scenarios concise but executable. Use exact commands only when verified from the repository. If a precondition or expected value is unknown, state `TBD` and identify who or what must confirm it.

### 6.3 Test location

Follow the repository's existing test structure. Prefer a nearby test module or package that covers the same component. Before proposing a new path, inspect adjacent tests and naming conventions. For multi-repository features, state which repository owns each test and why. Do not impose a generic directory layout on an established project.

## 7. Output

### 7.1 Use an existing format when available

Look for existing test plans and templates in the supplied repositories. If a relevant convention exists, follow it. Otherwise use the default structure below. Do not add a standards-conformance report unless the user requests one or the project template requires it.

### 7.2 Default structure

```markdown
# <Feature> Test Plan

**Status:** Draft
**Scope:** <feature / component>
**Repositories:** <repositories inspected>
**Baseline:** <branch and commit, when available>
**Source documents:** <requirements and design documents>

## 1. Overview
## 2. Scope and System Context
## 3. Requirements Assessment
## 4. Requirements Traceability
## 5. Test Strategy
## 6. Test Environment and Data
## 7. Test Scenarios
## 8. Existing Test Coverage and Gaps
## 9. Entry and Exit Criteria
## 10. Risks and Open Questions
## 11. Deferred Scenarios
## Appendix: References and Abbreviations
```

Include only applicable sections. Keep unresolved requirements and limitations visible.

### 7.3 Traceability matrix

Use a table with these columns where applicable:

| Requirement | Design reference | Component | Repository / code location | Existing test | Planned test IDs | Coverage |
|---|---|---|---|---|---|---|

### 7.4 Scenario summary

Provide a summary table before detailed steps:

| ID | Scenario | Priority | Level | Repository | Existing coverage | Implementation |
|---|---|---|---|---|---|---|---|

Detailed scenario example:

```markdown
### TC-001 — Reject a request with an invalid credential

- **Priority:** P1 — authorization boundary
- **Type / level:** Negative / API
- **Requirements:** REQ-003
- **Design:** `docs/design/auth.md` — Authentication
- **Repository:** `services/api`
- **Target test:** `tests/api/test_auth.py`
- **Existing coverage:** PARTIAL — invalid token is covered; expired token is not
- **Implementation:** TO ADD
- **Setup / data:** A test account and an expired test token

**Steps and expected results**
1. Submit a request using the expired token.
2. Verify the documented authentication error response.

- **Cleanup / dependencies:** None
```

The example illustrates format only. Use project-specific values only when supported by inspected sources.

### 7.5 Entry and exit criteria

Use project-defined criteria when available. Otherwise, label proposed criteria as `Proposed — requires team review`. Do not present them as existing policy. Include only criteria relevant to the feature, such as required scenario execution, unresolved critical defects, compatibility targets, or defined performance/reliability targets.

### 7.6 Output path

If the user asks for a file, follow the supplied path or repository convention. If no convention exists, use `docs/test-plans/<feature>-test-plan.md` when saving in the repository. Do not overwrite an existing plan unless requested; use a candidate filename and summarize the difference. If the user asks for the plan in chat, do not create a file.

## 8. Special cases

### 8.1 Requirements and design are available, but code is not

Create a **Preliminary Test Plan**. Derive scenarios from documented behavior and design. Mark code locations, existing coverage, and automation feasibility as `TBD` or `UNKNOWN`. State that code-level mapping remains to be done.

### 8.2 Code is available, but requirements or design are incomplete

Create a bounded plan using documented behavior and code observations. Label scenarios derived only from implementation as regression candidates. Do not treat undocumented implementation behavior as confirmed product intent.

### 8.3 Multiple repositories

Map component ownership and cross-repository contracts. Place tests at the most appropriate level and identify which repository owns each test. Use integration or E2E tests for behavior that cannot be verified within one component.

### 8.4 Test-only changes

When the supplied scope is limited to test code, produce a **Test Coverage Review** rather than a product test plan. Review what the tests assert, the product behaviors they target, and gaps in the tests themselves.

### 8.5 Documentation-only changes

Review consistency with requirements, design, and implementation. Include runtime scenarios only when the documentation changes a normative requirement, API contract, configuration, or operational procedure.

### 8.6 Change comparison

When the user supplies a base commit or comparison reference, use it to identify changed behavior and affected tests. Without a comparison reference, analyze the supplied repository state; do not assume all current code is part of the feature change.

## 9. Updating an existing plan

When updating a plan:

1. Re-read the source documents and current repository state.
2. Compare document versions/content and repository baselines when available.
3. Preserve existing IDs for scenarios that remain materially unchanged.
4. Add scenarios for new or changed requirements. Mark removed or superseded scenarios with a reason.
5. Reassess existing test coverage and update references.
6. Preserve human-authored notes unless they are demonstrably obsolete; flag uncertain content rather than deleting it.
7. Write to a candidate file unless the user explicitly requests an in-place update.
8. Summarize added, changed, removed, and unchanged scenarios.

## 10. Scope and safety

During test-plan generation:

- Do not query or depend on an issue tracker.
- Do not modify application code, test code, CI configuration, deployment configuration, or infrastructure.
- Do not install dependencies, deploy services, provision infrastructure, or run tests unless the user requests those actions.
- Do not run destructive, production-impacting, or environment-changing commands.
- Do not include credentials, tokens, private keys, or sensitive customer data in the plan. Use placeholders for secrets.
- Do not claim complete repository or test coverage unless the full relevant scope was inspected.
- Do not claim tests passed unless execution results were observed.

## 11. Final review checklist

Before presenting the plan, check that:

- [ ] The inspected documents and repositories are listed.
- [ ] Scope, assumptions, and limitations are stated.
- [ ] Requirements have stable IDs and source references.
- [ ] Ambiguous, unmeasurable, and conflicting requirements are visible.
- [ ] Relevant design elements and code locations are mapped where possible.
- [ ] Existing tests were checked by reviewing assertions, not filenames alone.
- [ ] Each testable requirement maps to a scenario or has an explanation.
- [ ] Each scenario has an expected result, priority, level, traceability, and coverage status.
- [ ] Test paths and commands are supported by repository evidence.
- [ ] Recommendations are distinguished from confirmed project requirements and capabilities.
- [ ] No existing plan was overwritten without permission.
