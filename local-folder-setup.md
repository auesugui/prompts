# Scaffold `_local/` Convention for Personal Agent Workflows

## Context

This repository is a shared UI codebase with multiple developers. We need a convention that allows individual developers to create personal agent workflow files (scenarios, HOPs, scratch prompts) without polluting the shared repo. These files are gitignored and never pushed.

## Requirements

1. **Add gitignore rule**: Append `**/_local/` to the project's `.gitignore` so that any `_local/` folder at any depth is excluded from version control.

2. **Create a `_local/` directory** at the project root with the following structure:

```
_local/
├── README.md
├── scenarios/
│   └── .gitkeep
└── hops/
    └── .gitkeep
```

3. **Create `_local/README.md`** with the following content:

```markdown
# _local/ — Personal Agent Workflows

This directory is **gitignored** and exists only on your local machine. Use it to store personal agent workflow files that are specific to how you work — scenarios, higher-order prompts (HOPs), scratch prompts, or any agent orchestration artifacts.

## Structure

- `scenarios/` — Markdown files describing step-by-step user workflows for agent-driven validation. Each file targets a specific page or flow in the application.
- `hops/` — Higher-order prompts that wrap scenario files in consistent execution patterns (setup, screenshot cadence, reporting, cleanup).

## Convention

- Any folder named `_local/` at any depth in this repo is gitignored.
- You can also create `_local/` folders inside specific directories (e.g., `src/components/_local/`) for colocated personal workflows.
- These files are for your local agent tooling (Copilot, Claude Code, etc.) and are never shared.

## Example Scenario File

```markdown
# Grid Sorting Validation

## Target
http://localhost:3000/orderbook

## Steps
1. Navigate to the orderbook grid
2. Click the "Price" column header to sort ascending
3. Verify rows are sorted by price ascending
4. Click "Price" column header again to sort descending
5. Verify rows are sorted by price descending
6. Click "Volume" column header
7. Verify rows are sorted by volume ascending
```

## Example HOP File

```markdown
# Debug UI Workflow

## Arguments
$1 = path to scenario file

## Workflow
1. Launch browser against target URL from scenario
2. Load and parse steps from $1
3. Execute each step, capture screenshot after each
4. If any step fails, capture DOM state + console errors
5. Save all artifacts to _local/output/{timestamp}/
6. Summarize: passed/failed steps, screenshot paths, errors
```
```

4. **Do not create any sample scenario or HOP files** beyond what's in the README examples. The developer will create their own.

5. **Verify** the `.gitignore` rule is working by confirming `git status` does not show any `_local/` files as untracked.
