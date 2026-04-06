# Agent Task: Set Up `_local/` Folder Convention

## What You're Doing

You are setting up a `_local/` folder convention in this repository. This creates a gitignored personal workspace where individual developers can keep scratch files, experiments, drafts, and local-only content without polluting the shared repository. This is infrastructure — keep it simple.

## Steps

### 1. Create the `_local/` directory at the repository root

```
_local/
```

Add a `.gitkeep` file inside so the directory structure is visible in the repo even though contents are ignored.

### 2. Update `.gitignore`

Add the following rule to the repository's `.gitignore` file. Place it in a clearly labeled section so its purpose is obvious:

```
# Personal workspace — local experiments, drafts, scratch files
# Each developer's _local/ content is gitignored and never committed
_local/*
!_local/.gitkeep
!_local/README.md
```

This ignores everything inside `_local/` except the `.gitkeep` (to preserve the folder) and the `README.md` (to document the convention).

### 3. Create `_local/README.md`

Write a brief README that explains:

- **What this folder is**: A gitignored personal workspace for each developer
- **What goes here**: Scratch files, personal skill drafts, experimental Playwright scenarios, local test fixtures, debugging notes, anything you don't want to commit but want to keep alongside the repo
- **What does NOT go here**: Nothing that the team depends on. If another developer needs your file, it doesn't belong in `_local/` — promote it to the shared repo
- **Convention**: This folder is gitignored so its contents are invisible to the rest of the team. You can organize it however you want internally. If you want sub-folders like `_local/playwright/`, `_local/skill-drafts/`, `_local/scratch/`, go for it

Keep the README concise — 10-15 lines max. Developers will read it once and then never look at it again. That's fine.

### 4. Verify

After setup, confirm:
- `git status` does not show any `_local/` contents (except README.md and .gitkeep)
- Creating a test file in `_local/test.txt` does not appear in `git status`
- The `.gitkeep` and `README.md` ARE tracked

### 5. Commit

Commit the changes with a conventional commit message:

```
chore: add _local/ folder convention for personal developer workspaces
```

## That's it

Do not over-engineer this. It's a folder and a gitignore rule. The value is in the convention existing, not in the implementation being clever.
