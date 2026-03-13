You are performing a multi-phase domain knowledge extraction from this workspace. The goal is to infer the business domain, user workflows, entity relationships, and application maturity of this Loans/Fixed Income syndicate trading platform — purely from reading the codebase.

## Why This Matters

This output will become a domain skill file — a reference document that future AI agents load to understand this application's business context when generating code, writing specs, or making architectural decisions. Accuracy matters more than completeness. If you're uncertain about something, flag it as an open question rather than guessing.

## Starting Point

Read the cross-repo-architecture file first:
- Location: [UI repo]/docs/cross-repo-architecture.md

This file is your table of contents. It describes the layers of the system: UI, Cache, Server, Proto, Domain, Liquibase, Utilities. Use it to understand how the repos connect before diving into any individual layer.

After reading it, produce an execution plan before doing any analysis. The plan should:

1. Identify the specific repos/directories you'll examine per phase
2. Order the phases by dependency (which layers inform understanding of other layers)
3. Estimate what each phase will reveal about the domain
4. Identify which phases can be combined vs. which need separation due to context limits

## Phased Extraction Framework

Use the following framework to structure your phases. Adapt the ordering and grouping based on what you learn from the architecture file — this is guidance, not a rigid sequence.

### Phase A: Data Model & Domain Vocabulary
**Target**: Liquibase migrations repo + Proto/protobuf definitions
**Extract**:
- Core domain entities (deals, facilities, commitments, allocations, positions, etc.)
- Entity relationships and cardinality (one-to-many, many-to-many)
- Field-level vocabulary — column names, enum values, status fields, type discriminators
- Schema evolution — what was added over time, what was renamed, what was deprecated
- Proto service definitions — what data crosses service boundaries, what fields are required vs optional

**Output format**:
```markdown
### Entities Identified
| Entity | Key Fields | Relationships | Confidence |
| --- | --- | --- | --- |

### Domain Vocabulary
| Term | Inferred Meaning | Source (table/proto/field) |
| --- | --- | --- |

### Status/Lifecycle Enums
| Entity | Status Field | Values | Inferred Workflow |
| --- | --- | --- | --- |

### Open Questions
- [ ] Items where meaning could not be reliably inferred
```

### Phase B: Business Logic & Workflows
**Target**: Server/Domain layer (Java/Kotlin Spring Boot services)
**Extract**:
- Service class responsibilities — what business operations does each service perform
- State transitions — trace how key entities move through lifecycle stages
- Validation rules — what business constraints are enforced in code
- Cross-service orchestration — which services call which, and in what order for key workflows
- Exception/error handling — what business error cases are explicitly handled (these reveal domain edge cases)
- Scheduled jobs or batch processes — these reveal operational workflows users may depend on

**Output format**:
```markdown
### Business Operations
| Operation | Service(s) | Input → Output | Business Purpose |
| --- | --- | --- | --- |

### Workflow State Machines
For each key entity lifecycle:
- States: [list]
- Transitions: [from → to, triggered by what]
- Guards/Conditions: [what prevents a transition]

### Validation Rules
| Rule | Entity/Field | Constraint | Business Rationale (inferred) |
| --- | --- | --- | --- |

### Open Questions
- [ ] Logic that exists but whose business purpose is unclear
```

### Phase C: User Experience & Feature Maturity
**Target**: UI layer (React/JavaScript)
**Extract**:
- Route structure — what pages/views exist, how they're organized
- Component hierarchy for key views (especially the orderbook grid)
- User roles/permissions — what auth guards exist, what role-based rendering occurs
- Form fields and their validations — these map directly to business data entry workflows
- Feature maturity assessment:
  - Mature: comprehensive error handling, loading states, edge case coverage, test coverage
  - Developing: happy path works, limited error handling, sparse tests
  - Skeletal: basic structure exists, TODOs present, minimal functionality
- Data fetching patterns — what API endpoints does the UI call, what data shapes does it expect
- User-facing labels and terminology — these are the business terms users actually see and use

**Output format**:
```markdown
### Application Views
| Route/View | Purpose | Key Components | Maturity |
| --- | --- | --- | --- |

### User Roles
| Role | Permissions/Access | Views Available |
| --- | --- | --- |

### Feature Maturity Matrix
| Feature | Maturity Level | Evidence |
| --- | --- | --- |

### UI Domain Vocabulary
| UI Label/Term | Maps to Entity/Field | Notes |
| --- | --- | --- |

### Open Questions
- [ ] Features whose intended user workflow is unclear from code alone
```

### Phase D: Synthesis & Domain Narrative
**Target**: All phase outputs combined
**Produce**:
- A coherent narrative describing this application as a product owner would explain it to a new developer
- Entity relationship map connecting all discovered entities
- End-to-end workflow descriptions for the primary user journeys
- A "what this application does NOT do" section — based on gaps, missing features, or boundaries you observed
- A confidence-rated summary: what you're highly confident about vs. what needs human verification
- A consolidated list of all open questions organized by who could answer them (PO, senior dev, business user)

## Handover Protocol

At the end of each phase, I will trigger a handover capture. When I do, structure your session state to fit this format, adapting the fields for a knowledge extraction context:

- **Session Summary**: Domain knowledge inferred this phase (not just "files I read" — the actual understanding gained)
- **What Worked**: Which directories/files were most informative for domain inference, which reading strategies produced the best signal
- **What Didn't Work**: Directories that were too large to process, files that were misleading or ambiguous, dead ends
- **Key Decisions**: Interpretation choices you made — e.g., "I interpreted 'allocation' to mean X based on Y evidence, but it could also mean Z"
- **Lessons Learned & Gotchas**: Non-obvious structural patterns, naming conventions, or code organization quirks that the next phase should know about
- **Current State**:
  | Aspect | Status |
  | Domain Model Coverage | % of entities mapped |
  | Workflow Coverage | % of key workflows traced |
  | Confidence Level | High / Medium / Low |
  | Knowledge Gaps | Summary of what's still unknown |
- **Next Steps**: What the next phase should prioritize based on what this phase revealed
- **Files Modified This Session**: In this workflow, this becomes "Files Created This Session" — the output docs produced
- **Open Questions**: Domain questions that can only be answered by a human with business context

## Critical Instructions

1. **Accuracy over completeness**. It's better to say "I found 5 entities I'm confident about" than to list 15 where half are guesses.
2. **Distinguish inference from assumption**. When you derive meaning from code, show your reasoning. "Field `pro_rata_share` in `allocation` table with DECIMAL type and FK to `facility` suggests this tracks each lender's proportional commitment to a facility" — that's inference with evidence. "This is a lending platform" without citing code is assumption.
3. **Flag ambiguity explicitly**. If a term, pattern, or workflow could be interpreted multiple ways, list the interpretations and what evidence would resolve it.
4. **Don't hallucinate domain knowledge**. You know general syndicated lending concepts from training data. Do NOT fill gaps in what you find in the code with generic industry knowledge. If the code doesn't show it, it doesn't go in the output. Generic knowledge goes in a separate "Expected but not found" section.
5. **Preserve the actual terminology**. Use the exact variable names, field names, and labels from the codebase. Don't normalize to industry-standard terms unless the code itself uses them.
6. **Track what you couldn't read**. If a file was too large, a directory too deep, or a pattern too complex to parse in context, log it. The human needs to know what the blind spots are.

## Begin

Start by reading the cross-repo-architecture file, then produce your execution plan. Do not begin analysis until the plan is confirmed.
