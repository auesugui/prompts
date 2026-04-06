# Agent Task: Design and Implement Validation Enforcement for Testing and Logging Skills

## Context — Read This First

This repository contains two recently created skills:

1. **Requirements-first testing skill** — defines how tests should be written with `@requirement` tags, the immutability rule (tests don't change unless requirements change), and a completion protocol the agent executes before reporting done.

2. **Structured logging skill** — defines a frontend logging convention, error boundary patterns, and a debugging protocol.

Both skills include inline completion protocols (Layer 1) that instruct the agent on what to check before reporting a task as done. These protocols work, but they are **probabilistic** — the agent may skip steps, especially under complex multi-file changes. The completion protocol is essentially a set of instructions the agent is supposed to follow, but there is no programmatic enforcement that guarantees it.

The goal of this task is to add **deterministic enforcement** — validation scripts and git hooks that run regardless of whether the agent followed the completion protocol. If the agent skips a step, the enforcement layer catches it before code gets committed.

## Architecture — Three Layers (Already Decided)

We use a three-layer validation model. Layer 1 already exists inside the skills. You are building Layers 2 and 3.

### Layer 1: Inline Acceptance Gates (EXISTS — do not modify)
These are the completion protocol sections inside each skill file. They instruct the agent to run checks before reporting done. They are the first line of defense but are not deterministic. Do not change these — they stay as-is.

### Layer 2: Per-Output-Type Validation Scripts (YOU ARE BUILDING THIS)
Standalone scripts that validate specific output types. Each script knows what "correct" looks like for its domain and can be invoked independently. These are the scripts that the Layer 1 completion protocols reference but that do not yet exist.

### Layer 3: Global Pre-Commit Gate (YOU ARE BUILDING THIS)
A git hook (pre-commit or pre-push) that runs the Layer 2 scripts automatically on every commit. This is the safety net — even if the agent completely ignores the completion protocol, the commit fails if validation doesn't pass.

## What to Scan Before Building

Before writing any scripts or hooks, gather context on:

1. **The two skill files**: Read both skills thoroughly. The testing skill's completion protocol and validation script reference section describe exactly what the scripts should check. The logging skill may have similar references. These are your requirements.

2. **Existing git hooks**: Check if this repo already has any pre-commit, pre-push, or other git hooks configured. Check for `.husky/`, `.git/hooks/`, `lint-staged` config in `package.json`, or any CI pipeline files that run validation. Document what exists so your implementation integrates with it rather than conflicting.

3. **CI/CD pipeline**: Examine any Jenkins, GitHub Actions, or other CI configuration. Understand what gates already run on PR or commit (linting, coverage checks, build verification). Layer 3 should complement these, not duplicate them.

4. **Test file conventions**: Look at how test files are structured — file naming patterns, describe block format, where test files live relative to source files. The validation scripts need to parse these reliably.

5. **The JIRA skill**: Read the JIRA skill to understand the acceptance criteria format. The validation scripts need to know what a valid `@requirement` tag looks like and how it maps to JIRA output.

6. **The qTest-to-Playwright skill**: Read this skill to understand how qTest-extracted tests are structured and where they live. The validation scripts need to distinguish between agent-written tests and qTest-extracted tests to enforce the no-duplication rule.

## Layer 2: Validation Scripts — What to Build

Design and implement standalone validation scripts. The testing skill's "Validation Script Reference" section documents the contract — what each script should check. Use that as your requirements spec.

At minimum, the following checks need to be enforced programmatically:

### Testing Validation

- **`@requirement` tag presence**: Every new or modified `describe` block in the diff must have a `@requirement` tag. Scan the git diff for added/modified test files, parse the describe blocks, confirm tags exist.
- **Immutability enforcement**: If a `describe` block that already has a `@requirement` tag was modified in the diff, check whether the `@requirement` or `@acceptance` tag content also changed. If the test assertions changed but the tag didn't, flag it — this means someone (or the agent) changed the test without updating the requirement. This is the most important check.
- **Coverage threshold verification**: Confirm that whatever coverage thresholds your CI enforces still pass. This may already be handled by existing CI — if so, don't duplicate it, just confirm it runs.
- **Tagged vs. untagged ratio**: Report (not block) the percentage of test describe blocks that have `@requirement` tags versus those that don't. This is a progress metric, not a gate.
- **No qTest duplication**: If a new Playwright e2e test covers a scenario that a qTest-extracted test already covers, flag it. This requires understanding where qTest-extracted tests live and comparing test descriptions or scenario IDs.

### Logging Validation

- **Error boundary coverage**: If a new React component was added in the diff, check whether it's wrapped in an error boundary (either directly or via a parent boundary). This may need to be a heuristic rather than exact — scan for error boundary usage patterns in the component tree.
- **Frontend logging utility usage**: If new API call handling code was added, check whether errors are logged using the structured logging utility defined in the logging skill (not raw `console.error`). This is enforceable by scanning for the utility's import/usage patterns.
- **Correlation ID propagation**: If new API integration code was added, check whether correlation IDs are captured from response headers and included in log entries. This may be enforceable via the API client wrapper if one exists.

### Implementation Decisions (For You to Make)

The following decisions depend on what you find in the codebase. Make the right call based on what's actually there:

- **Language**: Shell scripts? Node scripts? Whatever the repo's existing tooling supports. If the repo already uses a Node-based tool chain for linting and validation, write Node scripts. If it's shell-based, write shell scripts. Don't introduce a new paradigm.
- **Diff parsing**: How do you reliably parse the git diff to identify new/modified test files and describe blocks? Consider using `git diff --name-only` for file identification and AST parsing (via a tool like `ts-morph` or `jscodeshift` if Node-based) for describe block analysis. Regex on raw diff output is fragile — prefer AST parsing if the toolchain supports it.
- **Script location**: Where do validation scripts live in this repo? If there's an existing `scripts/` directory, use it. If not, create one with a clear naming convention.
- **Exit codes**: All scripts should exit 0 on pass and non-zero on failure, with clear error messages explaining what failed and how to fix it. The error messages should be actionable — not "validation failed" but "test file src/components/Allocation.test.tsx has a modified describe block without an updated @requirement tag. Either fix the code instead of the test, or update the @requirement tag to reflect the changed requirement."

## Layer 3: Pre-Commit Gate — What to Build

Design and implement a git hook that runs the Layer 2 validation scripts on every commit. Requirements:

- **Integration**: Use whatever hook mechanism the repo already uses (Husky, native git hooks, lint-staged, etc.). If nothing exists, recommend and implement the lightest-weight option.
- **Scope**: The hook should only run validation on files in the current commit (staged files), not the entire codebase. Nobody wants a 5-minute pre-commit hook.
- **Fail-fast**: If any Layer 2 script fails, the commit is blocked with a clear message. The developer can see exactly what failed and why.
- **Bypass**: Include a documented bypass mechanism (e.g., `git commit --no-verify`) for emergency situations, but make it clear in the error output that bypassing skips validation. This is intentional — sometimes you need to commit a WIP. But the bypass should feel deliberate, not default.
- **Performance**: The hook should add minimal time to the commit process. If AST parsing is slow, consider caching or only parsing changed files. The hook should not become something developers disable because it's annoying.

## Output

Produce:

1. **The validation scripts** (Layer 2) — placed in the appropriate directory with clear naming
2. **The git hook configuration** (Layer 3) — integrated with existing hook infrastructure or set up fresh
3. **A brief summary** documenting:
   - What each script checks and why
   - What existing infrastructure you integrated with vs. what you created new
   - Any checks from the skill completion protocols that could NOT be enforced programmatically and why (some checks may require human judgment — that's fine, document them as remaining Layer 1-only checks)
   - How to run the validation scripts manually (outside of the hook) for testing or debugging
   - The bypass mechanism and when it's appropriate to use

## Principles

- **Integrate, don't conflict.** If the repo already has linting, coverage checks, or CI gates, your scripts complement them. Don't duplicate what's already enforced.
- **Actionable errors.** Every failure message should tell the developer exactly what's wrong and how to fix it. "Validation failed" is not acceptable.
- **Incremental enforcement.** The scripts should not retroactively fail on existing untagged tests. They only enforce on new or modified code in the current diff. The frozen baseline principle from the testing skill applies here too.
- **The agent is the primary user.** These scripts will mostly be triggered by commits the agent makes. Design the error messages so the agent can read them and self-correct without human intervention. If a validation fails, the agent should be able to parse the error output and fix the problem.
