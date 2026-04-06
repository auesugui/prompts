# Agent Task: Build Two Skills for the Skills Repository

## What You're Doing

You are scanning this codebase and producing two complete skill files for our Skills Repository. These skills will govern how AI agents write tests and use structured logging on this application going forward. Read this entire prompt before taking any action.

## Background Context

Our test suite has been agent-written for the past 12-18 months with no design criteria beyond passing toll-gate requirements (100% line coverage, 85%+ branch coverage). Tests were written to hit numbers, not to encode requirements or prevent regressions. We use Jest for unit/component tests and Playwright for e2e.

Our LOB recently introduced a JIRA prompt template that produces clearly defined acceptance criteria. This means tickets going forward will have structured, agent-readable acceptance criteria. Older tickets in the backlog may not.

We are also missing a structured logging convention. When debugging, the agent has no documented schema for how logs are structured, what tags mean, or where to find them. This slows down debugging significantly.

## What to Scan

Before writing either skill, gather the following context from the codebase. Do not skip any of these steps.

### For the Testing Skill

1. **JIRA skill and prompt template**: There is an existing JIRA skill in this repository that handles ticket creation from the IDE. This skill contains references to the prompt template used to generate acceptance criteria. Read that skill file thoroughly — the acceptance criteria format it produces is the upstream input that the testing skill consumes. The testing skill's `@requirement` and `@acceptance` tags must map directly to the structure that the JIRA skill outputs. Do not design the testing skill's tagging convention in isolation — it must be compatible with what the JIRA skill already produces so the agent can programmatically derive tests from ticket output.

2. **Jest configuration and conventions**: Read `jest.config.*`, find 3-5 representative test files across different parts of the application. Document:
   - How describe blocks are structured
   - Naming conventions for test files (`.test.tsx`, `.spec.ts`, etc.)
   - What test utilities, custom renders, or shared fixtures exist
   - How mocking is handled (MSW? manual mocks? jest.mock?)
   - Where test files live relative to source files

3. **Playwright configuration and conventions**: Read `playwright.config.*`, find 2-3 representative e2e test files. Document:
   - How test suites are organized
   - What page objects or fixtures exist
   - How test data is managed
   - What selectors are used (data-testid? roles? css?)

4. **Current coverage enforcement**: Find any CI/CD configuration, pre-commit hooks, or pipeline files that enforce coverage thresholds. Document what gates exist today.

5. **Existing test patterns to avoid**: Identify 2-3 examples of tests that exist purely for coverage — tests that assert implementation details, snapshot tests with no behavioral assertion, or tests that just render and check "not null." These are the anti-patterns the new skill will steer away from.

### For the Logging Skill

The primary gap is on the frontend — backend logging exists but the frontend does not adequately capture, surface, or communicate errors. During e2e test sessions and UAT, UI white screens and application crashes provide no meaningful diagnostic information. The logging skill's focus is defining a frontend logging layer that integrates with the existing backend logging infrastructure.

Before designing the skill, scan the following:

1. **Domain skill**: There is a `p1loans-domain` skill (or similarly named) in this repository that documents the application's architecture and domain context. Read it thoroughly. It may reference logging infrastructure, service boundaries, API patterns, or observability tooling that informs the logging skill design.

2. **Microservices workspace**: Scan the workspace that contains the backend microservices. Document:
   - What logging framework is in use (likely Log4j or Log4j2 — confirm this)
   - What the current log format looks like (structured JSON? plain text? custom format?)
   - How correlation IDs are generated, propagated, and used across services
   - Whether logs are shipped to Splunk (likely — confirm and document how: direct integration? log files scraped by a forwarder? API?)
   - What log levels are used and whether there's a consistent convention or if it varies by service
   - Any existing tagging, MDC (Mapped Diagnostic Context), or structured metadata patterns
   - How the team currently queries or searches logs in Splunk (any saved searches, dashboards, or documented query patterns?)

3. **Frontend error boundary implementation**: Scan the React frontend codebase for:
   - All existing error boundary components (search for `componentDidCatch`, `getDerivedStateFromError`, or any component with "ErrorBoundary" or "error-boundary" in the name)
   - What the current error boundaries render on failure (generic fallback? white screen? any error details?)
   - How granular the boundaries are (app-level only? route-level? component-level?)
   - Whether error boundaries capture or log any context (component stack, last user action, correlation ID of last request)
   - Whether different environments (dev, staging, prod) render different error output

4. **Frontend logging and API error handling**: Scan the React frontend for:
   - Any existing logging utilities, wrappers, or services beyond raw `console.log`/`console.error`
   - How API calls are made (Axios? fetch wrapper? custom client?) and how errors from API responses are handled
   - Whether correlation IDs from backend responses are captured or surfaced anywhere on the frontend
   - Whether any monitoring or observability tools are integrated (Datadog, Sentry, etc.)
   - How the frontend currently communicates with the backend — REST endpoints, GraphQL, WebSocket for real-time data, etc.

5. **Test session debugging workflow**: Look for any documentation, runbooks, or patterns related to how the team debugs issues found during e2e testing or UAT. This may be in Confluence links, README files, or the domain skill. Understanding the current workflow helps design a logging convention that fits into how people actually work.

## Skill 1: Requirements-First Testing

### Design Decisions (Already Made — Follow These)

**Requirement tagging convention**: Every new or modified test must include a `@requirement` tag that explicitly states the behavioral requirement being tested. Format this as a JSDoc-style comment block at the top of each `describe` block:

```typescript
/**
 * @requirement JIRA-1234: User can view allocation breakdown by lender
 * @acceptance When a deal is in allocation phase, the allocation table
 *             displays each lender's committed amount, final allocation,
 *             and percentage of total deal size.
 */
describe('AllocationBreakdown', () => {
  // tests here
});
```

Adapt the exact format to match what you find in the codebase's existing conventions, but the `@requirement` and `@acceptance` tags are mandatory.

**The immutability rule**: Tests with `@requirement` tags do not get modified unless the requirement itself changed. If a tagged test breaks after a code change and the requirement hasn't changed, the code is wrong — fix the code, not the test. If the requirement genuinely changed, update both the `@requirement`/`@acceptance` tags AND the test assertions, and state explicitly what changed and why.

**UAT-driven requirement changes**: UAT testing will surface feedback that affects acceptance criteria. The skill must define how the agent handles three distinct scenarios:

1. **New edge cases discovered in UAT**: These are additions, not modifications. The JIRA ticket gets new acceptance criteria appended (original criteria stay intact). The agent writes new test cases with new `@requirement` tags pointing to the added criteria. Existing tagged tests are not touched.

2. **Modifications to existing criteria**: The original assumption was wrong or the business changed direction. The JIRA ticket's acceptance criteria must be updated FIRST — with a note on what changed and why — BEFORE the agent modifies the corresponding `@requirement`/`@acceptance` tags and test assertions. The ticket is the source of truth. The test is a derivative. The agent never modifies a tagged test without a corresponding ticket update. If the JIRA skill supports ticket modification, use it. If it's creation-only, the developer must update the ticket manually before the agent proceeds with test changes.

3. **Bugs found in UAT that match existing criteria**: The feature doesn't work as specified. The ticket doesn't change, the test doesn't change, the code gets fixed. This is the immutability rule working as designed.

**Frozen baseline for existing tests**: Do NOT retroactively tag or refactor existing tests. They pass today, they maintain toll-gate numbers, leave them alone. When code covered by an existing untagged test is modified, THAT is when the old test gets replaced with a properly tagged requirements-first test. Natural attrition, not a rewrite.

**Old-format tickets**: If a JIRA ticket was not created through the JIRA skill (and therefore lacks structured acceptance criteria), extract the implicit requirement from the ticket description, state it explicitly in the `@requirement` tag, and confirm with the developer before writing tests. The JIRA skill's output format is the source of truth for what "structured" means — any ticket that doesn't match that format is old-format.

**Dual-track compliance**: The skill must maintain toll-gate compliance (coverage thresholds) while adding the requirements layer on top. These are not in conflict — requirements-first tests still generate coverage. But if the agent needs to add a test purely for coverage that doesn't map to a clear requirement, it should tag it as `@coverage-only` so the team can see the distinction.

**qTest-to-Playwright integration**: There is an existing skill in this repository that extracts qTest scenarios into Playwright e2e tests. This means there are three sources of test requirements the agent must be aware of:

- **JIRA skill** — defines what should be built (acceptance criteria)
- **qTest-to-Playwright skill** — defines what QA expects to validate (test scenarios extracted into Playwright)
- **This testing skill** — defines how the agent writes and maintains tests against both inputs

The testing skill must account for this by instructing the agent to:

1. Before writing new Playwright e2e tests for a feature, check whether qTest-extracted tests already cover that path. Read the qTest-to-Playwright skill to understand the extraction output format and where extracted tests live.
2. If qTest-extracted tests cover a scenario, tag them with `@requirement` references linking back to the corresponding JIRA criteria or qTest scenario ID. These are not undisciplined legacy tests — they were derived from documented QA scenarios and should be treated as first-class.
3. If gaps exist between qTest coverage and JIRA acceptance criteria, write new Playwright tests only for the gaps. Do not duplicate coverage that the qTest extraction already provides.
4. When UAT surfaces new edge cases (see UAT section above), check whether the qTest scenarios have also been updated. If they have, re-run the qTest-to-Playwright extraction for the affected scenarios before writing additional tests manually.

### Completion Protocol (Layer 1 — Inline Acceptance Gate)

The skill must end with a completion protocol section that the agent executes before reporting a task as done. This is not optional — it's part of the task definition. The protocol should check:

1. Every new or modified component has a corresponding test file with `@requirement` tags
2. Every `@requirement` test passes
3. No `@requirement` test was modified without a stated requirement change
4. The existing test suite still passes (toll-gate numbers hold)
5. If Playwright e2e tests are relevant to the feature, they exist and are tagged — check both agent-written and qTest-extracted tests for coverage before writing new ones
6. No duplication between agent-written e2e tests and qTest-extracted e2e tests for the same scenario
7. If the structured logging skill's frontend log buffer is available, Playwright e2e tests should assert that no unhandled ERROR-level entries exist in the log buffer after test execution (see logging skill for integration details)

### Validation Script Reference (Layer 2)

Include a section that describes what a `validate-tests.sh` script should check. Don't write the script in the skill — just document the contract so it can be built separately:

- Confirm `@requirement` tags exist on all new `describe` blocks in the diff
- Confirm no tagged tests were modified without an accompanying requirement-change comment
- Confirm coverage thresholds still pass
- Report the ratio of tagged vs untagged tests as a progress metric

### Skill File Format

Follow the existing skill file format used in this repository. Look at 2-3 existing skills and match their structure (frontmatter, sections, examples). The skill should be self-contained — an agent reading only this file should know exactly how to write tests on this codebase.

## Skill 2: Structured Logging Convention

### Design Decisions (Already Made — Follow These)

**Purpose**: The logging skill solves a specific problem: during e2e test sessions and UAT, UI crashes and white screens provide no diagnostic information. The team has to manually open dev tools, find errors, cross-reference correlation IDs with backend logs in Splunk, and piece together what happened. The skill defines a frontend logging layer that integrates with existing backend logging to make this process fast — both for human developers during test sessions and for the agent when asked to debug.

**The agent builds what the agent reads**: The logging conventions documented in this skill should be designed so that the agent can both PRODUCE correctly formatted logs when writing new code AND CONSUME/ANALYZE logs when debugging. The schema is bidirectional.

**Do not redesign backend logging.** Backend logging infrastructure already exists (likely Log4j/Log4j2 with Splunk). The skill documents what's there — frameworks, formats, correlation ID patterns, Splunk query conventions — so the agent can read and trace backend logs. It does NOT propose changes to backend logging conventions unless clear gaps are found (e.g., inconsistent log levels across services). The backend section of the skill is documentation, not prescription.

**Frontend is the primary build target.** Based on your scan of the frontend, design the following:

1. **Enhanced error boundaries**: Design an error boundary pattern that the agent uses when writing or modifying components. The error boundary should capture:
   - The error message and component stack trace
   - The correlation ID of the last API request (pull from whatever request client/context the frontend uses)
   - The last N user actions or state changes leading up to the crash
   - The current route/view
   
   In non-production environments, the error boundary should render a developer-facing diagnostic panel that displays all of this in one place — no dev tools required. In production, it renders a user-friendly fallback and silently reports the diagnostic payload.
   
   Base this pattern on what you found in the existing error boundary scan. If the current boundaries are app-level only, the skill should define a strategy for more granular placement (route-level at minimum, feature-level where practical). If boundaries exist but are generic, the skill should define how to enhance them incrementally — same natural attrition approach as the testing skill, not a rewrite.

2. **Frontend logging utility**: Define a lightweight, structured logging utility that the agent uses in all new frontend code. This is NOT a rewrite of existing console.log statements across the codebase. It's a utility that new code uses going forward. The utility should:
   - Support log levels (ERROR, WARN, INFO, DEBUG) with clear definitions for what each means in the frontend context
   - Automatically include correlation ID, timestamp, component/feature name, and user role (not PII)
   - Tag logs by domain (adapt to actual application domains found in the domain skill — likely allocation, syndication, deal management, etc.) and by layer (`[UI]`)
   - Maintain an in-memory log buffer of the last N entries (configurable, suggest 100 as default). This buffer survives navigation within the SPA but clears on full page refresh. During test sessions, the buffer can be dumped to console or to a diagnostic panel to see the event sequence leading up to a failure
   - Optionally flush to backend/Splunk for deployed environments (define the API contract but don't require this for v1)

3. **API error handling pattern**: Define how the agent handles API errors in new code. When a backend call fails:
   - Log a structured entry using the frontend logging utility, including: correlation ID from the response headers, endpoint, HTTP status, error payload summary, and the user action that triggered the request
   - Surface the error appropriately in the UI (not a white screen — a meaningful error state in the component)
   - If the error is unrecoverable, let it propagate to the error boundary which captures the full diagnostic context

**Correlation ID threading**: Correlation IDs already exist on the backend. The skill should document how they're propagated (likely via response headers or a request context) and define how the frontend captures and carries them. The frontend logging utility should automatically attach the current correlation ID to every log entry. When the agent is debugging and sees a frontend error log, the correlation ID is the bridge to tracing the issue through Splunk on the backend.

**Debugging protocol**: Include a section that tells the agent how to debug using this logging infrastructure. The protocol should be:
1. If a UI error is reported, check the frontend log buffer (or diagnostic panel output) for the error-level entry
2. Extract the correlation ID from that entry
3. Trace the correlation ID through backend logs (document how to query Splunk for a specific correlation ID — adapt to whatever query patterns the team uses today)
4. Narrow to the root cause by following the request through UI → API → service layers using the structured tags
5. If the issue is frontend-only (no backend error), use the log buffer's event sequence to identify the state change or user action that triggered the failure

**Connection to the testing skill**: The frontend logging utility enables a new class of test assertions. Playwright e2e tests can check the frontend log buffer after test execution and assert that no error-level entries were logged during the test run. The testing skill's completion protocol should reference this: "After e2e test execution, verify the frontend log buffer contains no unhandled ERROR entries." This turns every e2e test into both a functional test and an error detection mechanism.

### Skill File Format

Same as the testing skill — match existing skill format in this repository.

## Output

Produce two complete skill files:

1. `requirements-first-testing.md` (or whatever naming convention the repo uses)
2. `structured-logging.md`

Place them wherever new skills are staged in this repository. If there's a drafts or review directory, use that. Otherwise place them alongside existing skills.

After writing both files, provide a brief summary of:
- What context you found vs. what was missing
- Any assumptions you made where context was unavailable
- What the developer should review or confirm before these skills are considered final
