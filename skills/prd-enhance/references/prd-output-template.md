# Enhanced PRD Template

Use this template to assemble the final enhanced PRD output. Replace bracketed placeholders with actual content. Remove sections that are not applicable, but document why they were removed.

---

```markdown
# [Feature Name]: Enhanced PRD

**Author:** [Original author]
**Enhanced:** [Date] with Claude
**Status:** [Draft | In Review | Approved]
**Last Updated:** [Date]

---

## Executive Summary

[2-3 sentence overview: what we're building, why, and the expected impact. Write for an executive who has 30 seconds.]

## Problem Statement

### The Problem
[Clear, specific description of the problem. Who experiences it? When? How often?]

### Evidence
| Source | Finding | Confidence |
|--------|---------|------------|
| [Dovetail/Research] | [Key insight] | [High/Medium/Low] |
| [Preset/Analytics] | [Data point] | [High/Medium/Low] |
| [Support tickets] | [Pattern] | [High/Medium/Low] |
| [Competitive analysis] | [Observation] | [High/Medium/Low] |

### Impact of Inaction
[What happens if we don't solve this? Quantify where possible.]

## Goals & Success Metrics

### Primary Goal
[One sentence. What does success look like?]

### Metrics
| Metric | Current Baseline | Target | Measurement Method | Timeline |
|--------|-----------------|--------|-------------------|----------|
| [Primary metric] | [Current] | [Target] | [How measured] | [When] |
| [Secondary metric] | [Current] | [Target] | [How measured] | [When] |
| [Leading indicator] | [Current] | [Target] | [How measured] | [When] |

### Non-Goals
- [Explicitly out of scope item 1]
- [Explicitly out of scope item 2]

## Solution Alternatives Considered

### Option A: [Selected approach] (SELECTED)
- **Description:** [Brief description]
- **Pros:** [Key advantages]
- **Cons:** [Key disadvantages]
- **Why selected:** [Rationale]

### Option B: [Alternative approach]
- **Description:** [Brief description]
- **Pros:** [Key advantages]
- **Cons:** [Key disadvantages]
- **Why rejected:** [Rationale]

### Option C: [Alternative approach]
- **Description:** [Brief description]
- **Pros:** [Key advantages]
- **Cons:** [Key disadvantages]
- **Why rejected:** [Rationale]

## Competitive Context

| Competitor | Approach | Strengths | Weaknesses | Our Differentiation |
|-----------|----------|-----------|------------|-------------------|
| [Competitor 1] | [How they solve it] | [What works] | [What doesn't] | [How we differ] |
| [Competitor 2] | [How they solve it] | [What works] | [What doesn't] | [How we differ] |

## Scope

### In Scope
- [Feature/capability 1]
- [Feature/capability 2]
- [Feature/capability 3]

### Out of Scope
- [Excluded item 1] — [Why excluded]
- [Excluded item 2] — [Why excluded]

### Future Considerations
- [Phase 2 item that was intentionally deferred]

## Requirements

### Requirement 1: [Name]
**Description:** [What this requirement delivers]

**Acceptance Criteria:**
- **Given** [precondition], **When** [action], **Then** [expected result]
- **Given** [precondition], **When** [action], **Then** [expected result]

**Edge Cases:**
- [Edge case 1]: [Expected behavior]
- [Edge case 2]: [Expected behavior]

### Requirement 2: [Name]
**Description:** [What this requirement delivers]

**Acceptance Criteria:**
- **Given** [precondition], **When** [action], **Then** [expected result]

**Edge Cases:**
- [Edge case 1]: [Expected behavior]

[Repeat for each requirement]

### Requirement Dependencies

| Requirement | Priority | Depends On | MVP? |
|------------|----------|------------|------|
| [Req 1] | P1 (must-have) | — | Yes |
| [Req 2] | P1 (must-have) | Req 1 | Yes |
| [Req 3] | P2 (important) | Req 1 | No |
| [Req 4] | P3 (nice-to-have) | — | No |

**MVP path:** [List the P1 requirements in implementation order, respecting dependencies. Confirm they form a viable, shippable increment.]

## Risk Assessment

### Technical Risks
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [Risk 1] | [H/M/L] | [H/M/L] | [Plan] |

### Product Risks
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [Risk 1] | [H/M/L] | [H/M/L] | [Plan] |

### Execution Risks
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [Risk 1] | [H/M/L] | [H/M/L] | [Plan] |

### Assumptions Requiring Validation
- [ ] [Assumption 1] — Validation method: [How to test]
- [ ] [Assumption 2] — Validation method: [How to test]

## Stakeholder Map

| Stakeholder | Role | Engagement Level | Key Concerns |
|------------|------|-----------------|-------------|
| [Name/Team] | Approver | Must approve before dev | [Concerns] |
| [Name/Team] | Consulted | Input needed during design | [Concerns] |
| [Name/Team] | Informed | Notify of changes | [Concerns] |
| [Name/Team] | Impacted | Affected by change | [Concerns] |

## Technical Considerations

### Architecture
[High-level architecture notes, system interactions, data flow]

### API Contracts
[Key API endpoints, request/response shapes if applicable]

### Data Model Changes
[Schema changes, migrations, data considerations]

### Performance
[Latency targets, throughput requirements, scaling considerations]

### Security & Privacy
[Authentication, authorization, PII handling, compliance requirements]

## Design Considerations

### User Flow
[High-level user journey or link to design artifacts]

### Key States
- **Loading:** [Behavior]
- **Empty:** [Behavior]
- **Error:** [Behavior]
- **Success:** [Behavior]

### Accessibility
[WCAG requirements, keyboard navigation, screen reader support]

## Rollout Plan

### Strategy
[Feature flags, gradual rollout %, A/B test plan]

### Monitoring
[Key dashboards, alerts, success/failure signals]

### Rollback
[Rollback trigger criteria, rollback procedure]

## Appendix

### Client Research Insights
[Detailed findings from Dovetail or other research tools]

### Data Analysis
[Detailed analytics from Preset or other data sources]

### Consistency Matrix

| Problem | Goal | Metric | Requirement(s) | Acceptance Criteria |
|---------|------|--------|----------------|-------------------|
| [Problem 1] | [Goal it maps to] | [Metric that measures it] | [Req(s) that address it] | [AC that validates it] |
| [Problem 2] | [Goal it maps to] | [Metric that measures it] | [Req(s) that address it] | [AC that validates it] |

[Every problem should trace to at least one goal, metric, and requirement. Every requirement should trace back to a problem. Flag any orphaned items.]

### References
- [Link to design files]
- [Link to technical RFC]
- [Link to related PRDs]
- [Link to research projects]
```
