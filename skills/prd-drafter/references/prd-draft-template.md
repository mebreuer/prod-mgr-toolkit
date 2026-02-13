# PRD Draft Template

Use this template to assemble the PRD draft output. This is intentionally lighter than the prd-enhance output template — it captures enough for a real conversation and hand-off to prd-enhance, not implementation-ready detail.

Replace bracketed placeholders with actual content. Remove sections that are not applicable, but document why they were removed.

---

```markdown
# [Feature Name]: PRD Draft

**Author:** [User name] with Claude
**Date:** [Date]
**Status:** Draft — ready for enhancement
**Input Type:** [Sentence / Pitch / Research Notes / Other]

---

## Executive Summary

[2-3 sentences: what we're proposing, why it matters, and the expected impact. Write for someone with 30 seconds.]

## Problem Statement

### The Problem
[Clear, specific description of the problem. Who experiences it? When? How often? What's the current workaround?]

### Evidence
| Source | Finding | Confidence |
|--------|---------|------------|
| [User research / Web search / Analytics / User input] | [Key insight] | [High / Medium / Low — be honest] |

**Confidence note:** [Acknowledge what's validated vs. assumed. E.g., "Problem validated through user interviews; frequency is estimated."]

### Impact of Inaction
[What happens if we don't solve this? Who suffers? What opportunity do we miss?]

## Target Users

### Primary User
- **Who:** [Role / segment]
- **Goal:** [What they're trying to accomplish]
- **Current Workflow:** [How they do it today]
- **Key Friction:** [Where it breaks down]

### Secondary User(s)
- **Who:** [Role / segment]
- **Relationship to primary:** [How they interact with or benefit from the solution]

## Goals & Success Metrics

### Primary Goal
[One sentence. What does success look like?]

### Metrics
| Metric | Current Baseline | Target | Notes |
|--------|-----------------|--------|-------|
| [Primary metric] | [Current or "Unknown"] | [Target or "TBD"] | [How we'd measure this] |
| [Secondary metric] | [Current or "Unknown"] | [Target or "TBD"] | [How we'd measure this] |
| [Leading indicator] | [Current or "Unknown"] | [Target or "TBD"] | [How we'd measure this] |

## Proposed Solution

### Experience Overview
[Describe the solution from the user's perspective — what they see, what they do, what changes for them. No technical details. 3-5 sentences.]

### Key Capabilities
- [Capability 1: what the user can do]
- [Capability 2: what the user can do]
- [Capability 3: what the user can do]

## Alternatives Considered

| Approach | Description | Pros | Cons | Why Not Selected |
|----------|-------------|------|------|-----------------|
| [Selected approach] | [Brief] | [Key pros] | [Key cons] | **Selected** |
| [Alternative 1] | [Brief] | [Key pros] | [Key cons] | [Reason] |
| [Alternative 2] | [Brief] | [Key pros] | [Key cons] | [Reason] |

## Scope

### In Scope
- [Capability / behavior 1]
- [Capability / behavior 2]
- [Capability / behavior 3]

### Out of Scope
- [Excluded item 1] — [Why]
- [Excluded item 2] — [Why]

## Requirements

High-level requirements per capability. Detailed acceptance criteria (Given/When/Then) are deferred to the enhancement phase.

### [Capability 1]
- [Requirement 1.1]
- [Requirement 1.2]
- [Requirement 1.3]

### [Capability 2]
- [Requirement 2.1]
- [Requirement 2.2]

### [Capability 3]
- [Requirement 3.1]
- [Requirement 3.2]

## Competitive Context

_Include if research was conducted. Remove if not — don't fabricate._

| Competitor / Alternative | Approach | Key Takeaway |
|-------------------------|----------|-------------|
| [Competitor 1] | [How they solve it] | [What we can learn] |
| [Competitor 2] | [How they solve it] | [What we can learn] |
| [Manual workaround] | [What users do today] | [What this tells us about the problem] |

## Open Questions

- [ ] [Unresolved question 1]
- [ ] [Unresolved question 2]
- [ ] [Unresolved question 3]

## Next Steps

### Enhancement Readiness
This draft is ready for hand-off to the **prd-enhance** skill, which will:
- Challenge and validate the problem statement with deeper exploration
- Run expert reviews (Product, Engineering, Design perspectives)
- Add client insights from Dovetail and data from Preset
- Detail requirements with acceptance criteria and edge cases
- Build a traceability matrix and risk assessment
- Produce an implementation-ready PRD

**To enhance:** Share this document with Claude and say "enhance this PRD."

### Other Next Steps
- [ ] [Any immediate action items from the drafting session]
- [ ] [Stakeholders to share the draft with]
- [ ] [Research to conduct before enhancement]
```
