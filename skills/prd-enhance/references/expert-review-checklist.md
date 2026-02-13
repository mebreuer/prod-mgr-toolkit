# Expert Review Checklist

A reference of review questions for evaluating a PRD from three expert perspectives. Use these to identify gaps, challenge assumptions, and strengthen the document.

## Product Leader Perspective

### Problem & Opportunity
1. Is the problem statement specific enough to be falsifiable? Could we prove this problem doesn't exist?
2. What evidence validates this is a real problem (not an assumed one)? Are we solving for actual user pain or internal intuition?
3. How large is the affected user segment? Is the opportunity sized with data or estimation?
4. What happens if we don't solve this? What's the cost of inaction?
5. Are we solving the root cause or a symptom? Could the real problem be upstream?

### Solution & Strategy
6. Why this solution over alternatives? What was explicitly considered and rejected?
7. Does this align with our current product strategy and roadmap priorities?
8. How does this affect existing user workflows? Are we introducing friction elsewhere?
9. What's the competitive landscape? Are we differentiating or catching up?
10. Is the scope right? Could we achieve 80% of the value with 20% of the effort?

### Success & Metrics
11. Are the success metrics measurable, time-bound, and tied to business outcomes?
12. What leading indicators will tell us early if this is working or failing?
13. What does failure look like, and what's our exit criteria for killing or pivoting?

## Engineering Leader Perspective

### Feasibility & Architecture
14. Is this technically feasible within the proposed timeline? What are the unknowns?
15. What existing systems does this touch? Are there hidden dependencies or migrations?
16. Does this introduce technical debt? Is there a plan to address it?
17. Are there performance implications (latency, throughput, data volume)?
18. What's the data model impact? Are schema changes required?

### Implementation & Quality
19. Are requirements specific enough for a developer to implement without clarifying questions?
20. Are edge cases documented? What happens with empty states, errors, timeouts, concurrent access?
21. Is the API contract defined? Are request/response shapes, error codes, and versioning specified?
22. What's the testing strategy? Are there scenarios that are hard to test?
23. Are there security or privacy implications? Does this handle PII, authentication, or authorization changes?

### Operations & Scale
24. What's the rollout strategy? Feature flags, gradual rollout, or big bang?
25. How do we monitor this in production? What alerts and dashboards are needed?
26. What's the rollback plan if something goes wrong?

## Design Leader Perspective

### User Experience
27. Is the user journey mapped end-to-end, including entry points and exit points?
28. How does this affect existing navigation and information architecture?
29. Are accessibility requirements defined (WCAG level, screen reader support, keyboard navigation)?
30. What are the key interaction states (loading, empty, error, success, partial)?

### Research & Validation
31. Has this been validated with users? What research supports the proposed UX?
32. Are there usability risks? What's the plan for user testing before/after launch?

### Consistency & Polish
33. Does this follow existing design system patterns, or does it require new components?
34. Is the content strategy defined? Who writes the copy, and is it reviewed for voice/tone?
35. What are the responsive/cross-platform considerations?

## Cross-Cutting Concerns

### Consistency & Traceability
43. Does each metric directly measure progress on the stated problem?
44. Does each requirement directly contribute to a stated goal?
45. Do acceptance criteria actually test the requirement, not adjacent functionality?

### Stakeholders
36. Who needs to approve this before development starts?
37. Who needs to be consulted during design/development (legal, compliance, support, marketing)?
38. Who needs to be informed about the change (internal teams, external partners)?
39. Who is impacted by this change but has no decision-making authority?

### Risk
40. What assumptions are we making that, if wrong, would invalidate the entire approach?
41. What are the top 3 risks, and what's the mitigation plan for each?
42. Are there regulatory, legal, or compliance considerations?
