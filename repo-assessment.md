You are analyzing a Skills Repository that contains agentic skills and reference files used by engineers and product owners to enhance Copilot workflows. The repo has documented contribution standards in CONTRIBUTING.md and uses a skill-creator pattern. I am the maintainer; I need your analysis to design an automated validation layer.
Your analysis will happen in four phases. Complete each phase fully before moving to the next, and output your findings at each phase so I can verify your understanding before you proceed.
Phase 1: Repository State Analysis
Before reading any specific PR, understand the current state of the repo.

Read CONTRIBUTING.md in full. Extract every explicit rule, convention, and pattern. Categorize each as:

Mechanical (can be checked by regex/lint: naming, file paths, required frontmatter fields)
Structural (requires parsing/understanding: file size thresholds, progressive disclosure patterns, skill-vs-reference distinction)
Judgment-based (requires taxonomic reasoning: directory placement, whether a skill overlaps existing skills, whether a new directory is warranted)


Walk the current repo structure. Produce:

Directory taxonomy with brief purpose of each top-level directory
Count of skills vs references vs other file types
File size distribution for skill files (min, max, median, p90)
List any existing skills or files that currently violate CONTRIBUTING.md — this is important for calibration. If the rules are violated elsewhere, that changes how we should enforce them on new contributions.


Identify conventions that are practiced in the repo but not documented in CONTRIBUTING.md. These are implicit standards that contributors can't be expected to know without reading the code. Flag these as documentation gaps.

Output Phase 1 findings before proceeding.
Phase 2: PR Analysis
I will provide a commit ID for a specific PR (Chirag's merge). Analyze it against the repo state you just mapped.
For each file added or modified in the PR:

Classification: Is this intended to be a skill, a reference, or something else? What does the file itself claim to be (frontmatter, naming, structure) vs. what its content suggests it actually is?
Mechanical compliance: Check against every mechanical rule from Phase 1. List passes and failures.
Structural compliance: Check against every structural rule. For size-related rules, if a file exceeds a threshold, assess whether progressive disclosure would be appropriate or whether the content genuinely needs to be single-file. Provide reasoning either way — don't just flag the violation.
Judgment-based concerns: For directory placement, assess whether the chosen location fits the existing taxonomy, whether an alternative existing directory would fit better, or whether a new directory would be warranted. For skill overlap, identify any existing skills this contribution duplicates or conflicts with.
Contribution pattern analysis: Did the contributor use the skill-creator pattern (if CONTRIBUTING.md requires it)? Is there evidence of manual creation where tooling should have been used?

Output Phase 2 findings as a structured table or list, one entry per file. Include a summary at the end categorizing total violations by type.
Phase 3: Validator Design Implications
Based on what you found in Phases 1 and 2, assess the design of an automated validator.

Which violations in this PR could have been caught by pure mechanical rules (pre-commit hook)? List them specifically. These are the cheapest wins.
Which violations required structural analysis (agent-based review)? List them specifically. For each, describe what the agent would need to know or reason about to catch it reliably.
Which violations required judgment calls where reasonable people could disagree? List them specifically. These are the cases where an exception-justification pattern makes sense — the agent flags them but doesn't block; the contributor must justify; a maintainer reviews the justification.
False-positive analysis: Are there any cases in this PR where strict automated enforcement would have blocked something legitimate? Be honest — a validator that cries wolf will be ignored or disabled. If the answer is "no false positives," say so; if there are edge cases, name them.
Gaps the validator cannot close: What patterns of non-conformance would still slip through even with a well-designed agent? These are the residual cases that require human maintainer review regardless.

Output Phase 3 findings.
Phase 4: Recommended Validator Architecture
Given everything above, recommend a specific validator architecture. Structure your recommendation as:

What runs at pre-commit time (cheapest, fastest, must be deterministic)
What runs at PR-open time (agent-based review, advisory comments)
What requires exception-justification from the contributor (agent flags, contributor justifies in PR, maintainer reviews justification only)
What requires full maintainer review regardless (things automation shouldn't touch)

For each layer, specify:

Concrete inputs and outputs
How exceptions are handled
How this layer would have performed on Chirag's PR specifically (caught it / flagged it / missed it)

End with a prioritized build order: what to ship first, second, third, and what to explicitly defer.

Operating principles throughout

Ground every claim in the actual repo state, not abstract best practices. If CONTRIBUTING.md doesn't require X, don't flag the absence of X as a violation — flag it as a documentation gap.
Distinguish "rule violation" from "aesthetic preference." I want the validator to enforce documented rules, not my taste.
Name your uncertainty. If you can't tell whether something is a skill or a reference from the file itself, say so — that ambiguity is itself a finding.
Treat the existing repo as the corpus of truth for implicit conventions. If 90% of skills follow pattern X and the PR violates pattern X, that's notable even if CONTRIBUTING.md is silent on it.

Begin with Phase 1. Do not skip ahead.
