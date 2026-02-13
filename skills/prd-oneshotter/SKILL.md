---
name: prd-oneshotter
description: >
  Use when given a PRD (with or without designs) to implement as a complete feature.
  Orchestrates codebase research, design decomposition or design discovery,
  gap analysis, tech spec creation, and agent team coding in one session.
  When no design exists, collaboratively defines screens and layouts using
  the existing design system components.
  Triggers: "build this feature", "implement this PRD", "here's the PRD and designs",
  "turn this spec into code", "one-shot this feature", "here's the PRD, no designs yet"
---

Announce: "I'm using the **prd-oneshotter** skill to turn this PRD + design into working code."

---

# PRD Oneshotter

> Spend more compute on research and questioning than on coding.
> A product manager should be able to trust the output without engineering knowledge.

## Overview

8-phase pipeline:
0. **Input Collection** — ingest PRD + design from any source
1. **PRD Digestion** — structured breakdown, confirm understanding
2. **Codebase Research** — 3 parallel agents discover components, patterns, data layer
3. **Design Decomposition** — label every element, map to design library; OR **Design Discovery** — collaboratively define screens using design library components when no design exists
4. **User Flow Mapping** — define every interaction, assume-then-verify
5. **Deep-Dive Questioning** — iterative, screen-by-screen questioning until all gaps are resolved
6. **Tech Spec** — produce implementation blueprint
7. **Team Spawn** — create agent team, implement, review

**Context management:** Each phase writes findings to files. Downstream phases read files — not inline context. This prevents context window exhaustion.

**Research files location:** `docs/plans/research/` in the working directory.

---

## Phase 0: Input Collection

### Accept PRD from any source

| Source | Method |
|--------|--------|
| Notion URL | Use `notion-fetch` MCP tool to retrieve content |
| PDF file path | Use `Read` tool |
| Local file (.md, .txt, etc.) | Use `Read` tool |
| Pasted text | Accept directly from the user message |
| Linear doc | Use Linear MCP `get_document` tool |

### Accept design from any source

| Source | Method |
|--------|--------|
| Figma URL | Parse `fileKey` and `nodeId`, use Figma MCP tools |
| Image files (.png, .jpg, etc.) | Use `Read` tool for visual analysis |
| Both Figma + images | Handle mixed input |
| No design provided | Enter **Design Discovery Mode** — collaboratively define screens, flows, and component composition using the design library (see Phase 3 alternate path) |

### Platform question

Immediately ask: **"Which platform should we build first — mobile or web?"**

Use `AskUserQuestion` with options: `Mobile (React Native)`, `Web (React)`, or let the user specify.

### Source Tracking

After collecting inputs, write `docs/plans/research/sources.md` recording:

```markdown
## Source Tracking

### PRD Source
- **Type:** [Notion URL | PDF file | Local file | Pasted text | Linear doc]
- **Location:** [URL, file path, or "inline"]

### Design Source
- **Type:** [Figma URL | Image files | No design (Design Discovery)]
- **References:** [Figma URL with fileKey/nodeIds | image file paths | "Design Discovery"]

### MCP Tools Used
- [ ] Sourcegraph
- [ ] Figma
- [ ] Notion
- [ ] Linear
```

This metadata is consumed by Phase 7 to determine which tool-loading instructions to include in agent prompts.

### Graceful Degradation

| Tool Failure | Fallback |
|-------------|----------|
| Notion MCP unavailable | Ask user to paste PRD content directly |
| Figma MCP unavailable | Ask user to provide screenshots instead |
| Linear MCP unavailable | Ask user to paste document content |
| PDF too large | Ask user to paste key sections |

---

## Phase 1: PRD Digestion

Parse the PRD into this exact structure:

```markdown
## PRD Summary

### Goals
- What success looks like (measurable outcomes)

### User Stories
- As a [who], I want [what], so that [why]

### Requirements
**Functional:**
- [Explicit from PRD]
- [Inferred from context — marked as inferred]

**Non-Functional:**
- Performance, security, accessibility requirements

### Constraints
- Technical, business, timeline constraints

### Success Metrics
- How we'll measure success

### Open Questions
- Anything ambiguous or missing from the PRD
```

Present this summary back to the user.

### Checkpoint 1

> **STOP.** Ask: "Does this capture the requirements correctly? Anything I missed or got wrong?"
>
> Do NOT proceed until the user confirms.

---

## Phase 2: Codebase Research (3 Parallel Agents)

### Memory check first

Before spawning agents, check for cached findings. If the memory file at `~/.claude/memory/prod-mgr-toolkit/prd-oneshotter/codebase-context.md` does not exist, create the directory and proceed with full discovery below. If it exists:

- **If file exists:** Present cached findings to user. Ask: "Is this context still accurate?"
  - If yes → skip or narrow research to only feature-specific queries
  - If partially → update the stale sections only
- **If file doesn't exist:** Proceed with full discovery below

### Create research output directory

```bash
mkdir -p docs/plans/research
```

### Spawn 3 Explore agents in parallel

Use the `Task` tool with `subagent_type: "Explore"` for all three. Each agent uses Sourcegraph MCP tools.

**IMPORTANT:** Before using Sourcegraph tools, load them first:
```
ToolSearch: "+sourcegraph"
```

#### Agent 1: Design Library Discovery

**Prompt must include:**
- "Find the component library / design system for this codebase"
- "Search for component exports, storybook, shared UI directories"
- "Catalog available components with their props/APIs"

**Tools to use:** `nls_search` ("design library components", "shared UI"), `keyword_search`, `list_files`, `read_file`

**Output file:** `docs/plans/research/design-library.md`

**Required output format:**
```markdown
## Design Library

**Repo:** [repo name]
**Path:** [directory path]

### Available Components

| Component | Import Path | Key Props | Notes |
|-----------|------------|-----------|-------|
| Button | @lib/ui/Button | variant, size, onPress | Primary/Secondary/Tertiary |
| Card | @lib/ui/Card | title, children | Supports header slot |
| ... | ... | ... | ... |

### Composition Patterns
- How components are typically composed in existing features
- Common layout patterns
```

#### Agent 2: Feature Pattern Research

**Prompt must include:**
- "Find existing features similar to [feature description from PRD]"
- "Document folder structure, file organization, state management patterns"
- "Find routing and navigation patterns"

**Tools to use:** `nls_search` (feature-specific queries), `keyword_search`, `read_file`

**Output file:** `docs/plans/research/feature-patterns.md`

**Required output format:**
```markdown
## Feature Patterns

**Repo:** [repo name]

### Folder Structure Convention
```
features/
  feature-name/
    components/
    hooks/
    screens/ (or pages/)
    types/
    utils/
    index.ts
```

### Similar Features Found
- [Feature A]: [path] — [what it does, why it's relevant]
- [Feature B]: [path] — [what it does, why it's relevant]

### Architectural Patterns
- State management: [approach]
- Data fetching: [approach]
- Navigation: [approach]
- Error handling: [approach]

### File Naming Conventions
- Components: [convention]
- Hooks: [convention]
- Types: [convention]
```

#### Agent 3: Data Layer Research

**Prompt must include:**
- "Find API clients, endpoints, data models relevant to [feature area]"
- "Document state management approach (Redux, Context, MobX, etc.)"
- "Find authentication/authorization patterns"
- "Find reusable hooks and utilities"

**Tools to use:** `keyword_search`, `find_symbol_definition`, `read_file`

**Output file:** `docs/plans/research/data-layer.md`

**Required output format:**
```markdown
## Data Layer

**Repo:** [repo name]

### API Patterns
- Client library: [e.g., Apollo, axios, fetch wrapper]
- Base URL config: [location]
- Auth headers: [how attached]

### Relevant Endpoints
| Endpoint | Method | Purpose | Request Shape | Response Shape |
|----------|--------|---------|---------------|----------------|
| /api/... | GET | ... | ... | ... |

### Data Models
- [Model A]: [location] — [fields summary]
- [Model B]: [location] — [fields summary]

### State Management
- Approach: [Redux/Context/MobX/etc.]
- Store location: [path]
- Relevant slices/contexts: [list]

### Reusable Hooks
| Hook | Location | Purpose |
|------|----------|---------|
| useAuth | ... | Authentication state |
| useFetch | ... | Data fetching wrapper |
```

### Knowledge synthesis

After all 3 agents complete, synthesize findings into a unified context document:

**Write to:** `docs/plans/research/codebase-context.md`

This document combines the key findings from all three research files into a single reference. Every downstream phase and implementation agent receives this file path.

### Memory update

Update `~/.claude/memory/prod-mgr-toolkit/prd-oneshotter/codebase-context.md` with **stable** findings only:
- Design library location and component list
- Architecture patterns and conventions
- State management approach
- File naming conventions

Do NOT cache feature-specific findings (those change per feature).

---

## Phase 3: Design Decomposition

This phase has two paths depending on whether designs were provided.

---

### Path A: Design provided (Figma or images)

#### If Figma URL was provided

Load Figma tools first:
```
ToolSearch: "+figma"
```

Execute in sequence:
1. `get_metadata` → lightweight XML structure of all layers
2. `get_screenshot` → visual reference image
3. `get_code_connect_map` → check for existing component mappings
4. `get_design_context` → extract code hints and styling

#### If images were provided

Use `Read` tool for visual analysis of each screenshot.

#### For EVERY visual element

Create a decomposition table:

| # | Element Label | Design Library Component | States | Interactions | Data Source |
|---|--------------|-------------------------|--------|--------------|-------------|
| 1 | Account Balance Card | `BalanceCard` | default, loading, error | tap → account detail | accountBalance API |
| 2 | Transaction List Item | `ListItem` | default, pending, failed | tap → transaction detail | transactions API |
| 3 | Quick Actions Bar | **NO MATCH** | default, disabled | tap per action → varies | feature flags |

**If an element has NO matching design library component:**

> **FLAG IT.** Ask the user: "This element ([label]) doesn't match any existing component. Should I:
> (a) Build a custom component following design library patterns?
> (b) Use the closest match ([component name]) and adapt it?
> (c) Skip this element for now?"

---

### Path B: No design provided (Design Discovery Mode)

When no design exists, YOU become the designer. Use the design library discovered in Phase 2 and the PRD from Phase 1 to collaboratively define what to build.

#### Step 1: Screen Inventory

From the PRD's user stories and requirements, identify every screen needed:

```markdown
### Screens Needed

| # | Screen Name | Purpose | Entry Point |
|---|------------|---------|-------------|
| 1 | [Screen A] | [What the user does here] | [How they get here] |
| 2 | [Screen B] | [What the user does here] | [From Screen A via...] |
```

Present this to the user: **"Based on the PRD, I think we need these screens. Am I missing any?"**

#### Step 2: Screen-by-Screen Composition

For each screen, propose a layout using ONLY design library components from Phase 2:

```markdown
### Screen: [Screen Name]

**Layout:**
- Header: `NavigationBar` (title: "[title]", leftAction: back)
- Section 1: `SectionHeader` (title: "[section]")
  - Content: `List` with `ListItem` rows (icon, title, subtitle, chevron)
- Section 2: `Card` (variant: elevated)
  - Inside: `BalanceDisplay` + `Button` (variant: primary, label: "[CTA]")
- Footer: `BottomBar` with `TabItem` entries

**Data requirements:**
- [What data drives each component]

**States:**
- Default: [description]
- Loading: [Skeleton/Shimmer using `SkeletonLoader`]
- Empty: [EmptyState component with message + CTA]
- Error: [ErrorState component with retry]
```

**For each screen, ask:** "Does this layout match what you had in mind? What would you change?"

#### Step 3: Identify gaps

After proposing all screens, flag anything the design library can't handle:

> "These UI elements don't have a matching design library component:
> 1. [Element] — closest match is [component], but it lacks [capability]
> 2. [Element] — no match at all
>
> Should I build custom components for these following design library patterns?"

#### Design Discovery output

Produce the same decomposition table as Path A, but built from collaboration rather than visual analysis:

| # | Element Label | Design Library Component | States | Interactions | Data Source |
|---|--------------|-------------------------|--------|--------------|-------------|
| 1 | [Element from composition] | [Component] | [States defined above] | [From Step 2] | [Data source] |

---

### Checkpoint 2

> **STOP.** Present the labeled design decomposition (from either Path A or Path B).
> Ask: "Are these labels and component mappings correct?"
>
> Do NOT proceed until the user confirms.

---

## Phase 4: User Flow Mapping

### Per-element flow

For every interactive element from Phase 3, define:

| Element | Action | Result | Destination | Confidence |
|---------|--------|--------|-------------|------------|
| Balance Card | Tap | Navigate | Account Detail screen | HIGH |
| Balance Card | Long-press | Copy balance | Clipboard + toast | LOW |
| Transaction Item | Tap | Navigate | Transaction Detail | HIGH |
| Transaction Item | Swipe left | Show actions | Inline action buttons | MEDIUM |
| Back Button | Tap | Navigate back | Previous screen | HIGH |

**Confidence levels:**
- **HIGH** — Obvious from design or PRD (stated explicitly)
- **MEDIUM** — Reasonable inference from patterns or conventions
- **LOW** — Guessing based on common UX patterns

### Screen-level flow

For each screen in the feature:

```markdown
### [Screen Name]

**Entry paths:**
- Tab bar → [tab name]
- Push from [screen] → [trigger]
- Deep link: [URL pattern]

**Exit paths:**
- Back button → [previous screen]
- Close button → [destination]
- Navigate forward → [next screen]

**State transitions:**
- Default → Loading (on mount)
- Loading → Loaded (data received)
- Loading → Error (request failed)
- Loaded → Refreshing (pull-to-refresh)
```

### Approach: Assume first, verify second

1. Make reasonable assumptions for ALL interactions
2. Mark confidence levels honestly
3. Present the complete flow table
4. Ask user to confirm/correct MEDIUM and LOW items
5. HIGH confidence items are stated but don't block

### Checkpoint 3

> **STOP.** Present the interaction map. Ask: "Here's what I think happens for each interaction. Correct anything I got wrong."
>
> Focus user attention on MEDIUM and LOW confidence items.
> Do NOT proceed until the user confirms.

---

## Phase 5: Deep-Dive Questioning

By now, user flows are mapped. This phase replaces a single batch of questions with **iterative, screen-specific questioning** that exhaustively resolves all gaps before building the tech spec.

> One round of questions is never enough. First-round answers reveal new information requiring follow-ups.

---

### Phase 5A: Screen-by-Screen Deep Dive

For **EACH screen** from Phase 3/4, generate targeted questions covering:

#### Per-section questions (for every visible section of each screen):
- **Data fields:** Which endpoint? Which response field? Any conditional logic or transforms?
- **Business logic:** Visibility rules, sorting, limits, computed values, formatting
- **States:** Exact default/empty/loading/error/partial-data rendering for THIS section
- **Conditional rendering:** User roles, feature flags, A/B tests, time/location-based rules

#### Presentation rules:
- Present **1-3 screens at a time** (not all at once)
- Group questions by screen, then by section within each screen
- Number questions sequentially across batches for easy reference

#### Checkpoint 4A

> **STOP** after each batch of 1-3 screens.
> Ask: "Here are the questions for [Screen A], [Screen B], [Screen C]. Please answer these before I move to the next screens."
>
> Do NOT present the next batch until the current batch is answered.

---

### Phase 5B: Flow-by-Flow Questioning

For **EACH user flow** (connected path through screens) from Phase 4:

- **Transition triggers:** Exact action that triggers it, animation type, cancellability
- **State carriage:** What data passes between screens? Source of truth for shared state?
- **Back-navigation:** Behavior on back, unsaved state prompts, refetch vs cache
- **Interruptions:** App backgrounding mid-flow, connectivity loss, push notifications
- **Completion/failure:** Success definition, mid-flow failure handling, save draft capability

#### Checkpoint 4B

> **STOP.** Present all flow questions.
> Ask: "Here are the questions about how screens connect and interact. Please answer these."
>
> Do NOT proceed until answered.

---

### Phase 5C: Confidence Resolution

Compile **ALL MEDIUM and LOW confidence items** from Phase 4 into a resolution table:

```markdown
## Confidence Resolution

| # | Item | Current Confidence | Screen/Flow | Question |
|---|------|--------------------|-------------|----------|
| 1 | Long-press on Balance Card → copy | LOW | Home Screen | Does long-press copy the balance, or should it open a context menu? |
| 2 | Swipe-left on Transaction → actions | MEDIUM | Transaction List | Which actions appear? Delete? Flag? Share? |
```

Every MEDIUM and LOW item MUST be either:
- **Confirmed** → upgraded to HIGH
- **Corrected** → updated with the user's answer and set to HIGH

#### Checkpoint 4C

> **STOP.** Present the confidence resolution table.
> Ask: "These items need confirmation before I can build the tech spec. Please confirm or correct each one."
>
> Do NOT proceed until **zero MEDIUM or LOW items remain.**

---

### Phase 5D: Iterative Deepening (repeat until done)

After each round of answers from 5A-5C:

1. **Parse answers for new implications** — does this answer create new questions?
2. **Cross-reference answers across screens/flows** — are any answers inconsistent?
3. **Identify new gaps** revealed by the answers
4. **Present follow-up questions** grouped by screen/flow

#### Termination criteria (ALL must be true):
- Every data field has a confirmed endpoint source
- Every MEDIUM/LOW confidence item resolved to HIGH
- Every conditional rendering rule has explicit conditions
- Every error/empty/loading state has defined behavior
- Every flow transition has confirmed trigger + state carriage + back-nav
- User explicitly says "no more questions"

#### Max rounds: 4
After 4 rounds of iterative deepening, document any remaining unknowns as `USER DECISION (low confidence)` and proceed.

#### Checkpoint 4D

> **STOP.** After termination criteria are met (or max rounds reached):
> Ask: "I believe all gaps are resolved. Here's a summary of remaining unknowns (if any). Ready to proceed to the tech spec?"
>
> Do NOT proceed until the user confirms.

---

### Phase 5 Output

Write the full Q&A transcript to `docs/plans/research/gap-analysis-qa.md`:

```markdown
## Gap Analysis Q&A

### Screen-by-Screen (Phase 5A)

#### [Screen Name]
**Q1:** [question]
**A1:** [user's answer]

**Q2:** [question]
**A2:** [user's answer]

[...]

### Flow-by-Flow (Phase 5B)

#### [Flow Name: Screen A → Screen B → Screen C]
**Q1:** [question]
**A1:** [user's answer]

[...]

### Confidence Resolutions (Phase 5C)

| # | Item | Resolution | Final Confidence |
|---|------|-----------|-----------------|
| 1 | Long-press on Balance Card | Opens context menu (copy, share) | HIGH |
| 2 | Swipe-left on Transaction | Shows Delete and Flag actions | HIGH |

### Iterative Follow-ups (Phase 5D)

#### Round [N]
**Q1:** [follow-up question triggered by previous answer]
**A1:** [user's answer]

[...]

### Remaining Unknowns
- [Item]: USER DECISION (low confidence) — [assumption made]
```

---

## Phase 6: Tech Spec Generation

Load the tech spec template:
```
Read references/tech-spec-template.md
```

Using the template, produce a complete tech spec. Write it to:
```
docs/plans/tech-spec.md
```

The tech spec pulls from:
- Phase 1: Requirements and constraints
- Phase 2: Codebase research (reference files by path)
- Phase 3: Component mappings
- Phase 4: User flow map
- Phase 5: Gap analysis Q&A (`docs/plans/research/gap-analysis-qa.md`)

### Implementation tasks

The tech spec MUST end with an ordered list of discrete implementation tasks. Each task should be:
- Small enough for one agent to complete in a focused session
- Independent enough to be worked on in parallel where possible
- Specific enough that an agent can implement without asking questions

### Checkpoint 5

> **STOP.** Present the tech spec (or a summary if very long, with link to full file).
> Ask: "Does this tech spec look right? Ready to build?"
>
> Do NOT proceed to implementation until the user approves.

---

## Phase 7: Team Spawn & Implementation

### Create the team

```
TeamCreate:
  team_name: "prd-oneshotter-[feature-name]"
  description: "Implementing [feature name] from PRD"
```

### Context injection rule

**CRITICAL:** Every implementation agent's prompt MUST include ALL 9 of the following:
1. **PRD context** — full content if short, Phase 1 summary if long, + source link from `docs/plans/research/sources.md`
2. **Design references** — Figma URLs with fileKey/nodeId, image paths, or Design Discovery note (from `docs/plans/research/sources.md`)
3. **Specific task** from the tech spec
4. **Relevant gap analysis Q&A entries** — filtered to this agent's screens/flows (from `docs/plans/research/gap-analysis-qa.md`)
5. **Design library components** — full table (from `docs/plans/research/design-library.md`)
6. **Architecture patterns** — full section (from `docs/plans/research/feature-patterns.md`)
7. **API endpoints** for this task (excerpted from `docs/plans/research/data-layer.md`)
8. **User flow map** for this task's elements
9. **Tool-loading instructions** — Sourcegraph always; Figma/Notion/Linear conditional on `docs/plans/research/sources.md`

**How to build agent prompts:**

The orchestrator (you) MUST read each research file and include the FULL relevant content in the agent prompt — not summaries, not file paths alone. Agents cannot inherit your context. They start from zero.

For each agent prompt:
1. Read `docs/plans/research/sources.md` — determine which tools this agent needs
2. Read `docs/plans/research/gap-analysis-qa.md` — copy Q&A entries **relevant to this agent's screens/flows**
3. Read `docs/plans/research/design-library.md` — copy the **full component table** into the prompt
4. Read `docs/plans/research/feature-patterns.md` — copy the **folder structure and patterns** into the prompt
5. Read `docs/plans/research/data-layer.md` — copy **relevant endpoints and models** for this agent's specific task
6. Read `docs/plans/tech-spec.md` — copy this agent's **specific task section** and **relevant user flows**

**Prompt template for implementation agents:**

```
You are implementing [task name] for the [feature name] feature.

## Why We're Building This (PRD Context)
[If PRD is short: copy full PRD content]
[If PRD is long: copy Phase 1 summary]
[Include source link from docs/plans/research/sources.md]

This is the product intent. Use it to make UX decisions that align with the product goals.

## Design References
[If Figma URL exists:]
  Figma file: [URL with fileKey/nodeIds]
  To check design details, load Figma tools: ToolSearch: "+figma"
  Then use: get_screenshot, get_design_context, get_metadata
[If image files:]
  Design screenshots: [file paths]
  Use Read tool to view these for visual reference.
[If Design Discovery:]
  No formal designs exist — screens were collaboratively defined in Phase 3.
  Refer to the component composition in the tech spec.

## Business Logic Decisions (from Gap Analysis)
[Copy relevant Q&A entries from docs/plans/research/gap-analysis-qa.md filtered to this agent's screens/flows]

⚠️ These are user-confirmed decisions. Do NOT deviate from these answers.
If you encounter a scenario not covered here, STOP and message the team lead.

## Your Task
[Copy the FULL task section from docs/plans/tech-spec.md — description, files, acceptance criteria, everything]

## Design Library (USE ONLY THESE COMPONENTS)
[Copy the FULL component table from docs/plans/research/design-library.md]
[Include repo name, directory path, import paths, and prop signatures]

If you need a component not listed here, STOP and message the team lead. Do NOT create custom components.

## Architecture Patterns (FOLLOW THESE CONVENTIONS)
[Copy folder structure, naming conventions, and architectural patterns from docs/plans/research/feature-patterns.md]

## Data Layer (API endpoints and models for your task)
[Copy relevant endpoints, data models, hooks, and state management patterns from docs/plans/research/data-layer.md]

## User Flows (interactions you're implementing)
[Copy the flow map rows from the tech spec that apply to elements in this task]

## Reference Files (read these if you need more detail)
- Full tech spec: docs/plans/tech-spec.md
- Design library catalog: docs/plans/research/design-library.md
- Feature patterns: docs/plans/research/feature-patterns.md
- Data layer: docs/plans/research/data-layer.md
- Codebase context: docs/plans/research/codebase-context.md
- Gap analysis Q&A: docs/plans/research/gap-analysis-qa.md

## Research Tools
Use these to look up patterns, definitions, or details you encounter during implementation.

**Sourcegraph (always available):**
Load tools: ToolSearch: "+sourcegraph"
- `nls_search` — natural language search for concepts (e.g., "how is authentication handled")
- `keyword_search` — exact keyword/symbol search (e.g., "useAccountBalance")
- `read_file` — read a specific file from the repo
- `go_to_definition` — jump to where a symbol is defined
- `find_references` — find all usages of a symbol

[If Figma was used (from sources.md):]
**Figma (design reference):**
Load tools: ToolSearch: "+figma"
- Use to check design details, spacing, colors, or component structure

[If Notion was used (from sources.md):]
**Notion (PRD source):**
Load tools: ToolSearch: "+notion"
- Use to re-read PRD sections if you need more context

[If Linear was used (from sources.md):]
**Linear (original doc):**
Load tools: ToolSearch: "+linear"
- Use to check original requirements or linked documents

## Rules
- ONLY use components from the design library listed above — no exceptions
- Follow the folder structure and naming conventions exactly as documented
- If you need a component that doesn't exist in the design library, STOP and message the team lead — do NOT invent one
- Business logic decisions from the Gap Analysis section are FINAL — do not re-interpret or change them
- Read the reference files above if you need more context on any pattern or component
- Use Sourcegraph to look up any pattern, hook, or utility you encounter during implementation
- Include tests for your implementation
- Commit your work when the task is complete
```

**Reviewer agent prompt must additionally include:**
- The full component table (to verify components used are from the design library)
- The full tech spec (to verify implementation matches requirements)
- The full gap analysis Q&A (to verify business logic decisions were followed)
- Instruction: "For each file, verify: (1) all imported components exist in the design library, (2) folder structure matches conventions, (3) tests exist and cover the acceptance criteria, (4) business logic matches gap analysis decisions — flag any contradictions"
- Sourcegraph tool-loading instructions: `ToolSearch: "+sourcegraph"` with usage examples for verifying patterns against the codebase

### Team structure

| Role | Agent Type | Count | Responsibility |
|------|-----------|-------|---------------|
| Team Lead | general-purpose | 1 | Task management, blocker resolution, final review |
| Implementer | general-purpose | 2-3 | One per feature area from task list |
| Reviewer | general-purpose | 1 | Validates against tech spec and design library compliance |

### Workflow

1. Create team with `TeamCreate`
2. Create tasks with `TaskCreate` (from tech spec task list)
3. Set up task dependencies with `TaskUpdate` (ordered tasks block later ones)
4. Spawn implementer agents with `Task` tool, `team_name` parameter set
5. Each implementer: implement → test → self-review → notify team lead
6. Reviewer validates each completed task:
   - Component usage matches design library catalog?
   - Follows architectural patterns from research?
   - Tests pass?
   - Matches tech spec requirements?
7. Team lead resolves reviewer feedback
8. Repeat until all tasks complete
9. Final integration test
10. Shutdown team with `SendMessage` type: "shutdown_request"

### Implementation rules for agents

- **ONLY** use components from the design library (the exact list is in their prompt)
- **Follow** existing codebase patterns (the exact patterns are in their prompt)
- Each task MUST include tests
- Each task MUST be committed separately
- If an agent can't find a design library component for something, it MUST **STOP** and ask the team lead — do NOT invent custom components

---

## Common Mistakes

| Mistake | Why It's Wrong | What to Do Instead |
|---------|---------------|-------------------|
| Skipping Phase 2 research | Agents will use random components, wrong patterns | Always research first — even if it takes time |
| Asking too few questions in Phase 5 | Gaps become bugs during implementation | Minimum 3 questions per category, no exceptions |
| Passing context inline instead of via files | Context window exhaustion in Phase 7 | Write to files, pass file paths |
| Letting agents invent components | Inconsistent UI, doesn't match design system | Strict design library enforcement |
| Skipping checkpoints | User loses trust, wrong assumptions compound | Every checkpoint is mandatory |
| Caching feature-specific findings in memory | Stale data on next run | Only cache stable patterns (library location, conventions) |
| Not including gap analysis Q&A in agent prompts | Agents re-make decisions the user already answered | Copy relevant Q&A entries into every agent prompt |
| Not giving agents tool access (Sourcegraph, Figma) | Agents can't look up patterns they encounter during implementation | Include tool-loading instructions based on sources.md |
| Not including PRD context in agent prompts | Agents don't understand product intent, make wrong UX decisions | Include full PRD or Phase 1 summary + source link |

## Rationalization Traps

| You Might Think | Reality |
|----------------|---------|
| "The PRD is detailed enough, I can skip questions" | PRDs always have gaps. Phase 5 exists because PRDs are never complete. |
| "I know this codebase, I can skip research" | Your cached knowledge may be stale. At minimum, check memory file. |
| "This component is close enough" | Close enough = inconsistent. Use exact matches or flag it. |
| "I'll ask questions as they come up during coding" | Questions during coding = context switches for the user. Batch them in Phase 5. |
| "The design is self-explanatory, skip flow mapping" | Designs show appearance, not behavior. Phase 4 defines behavior. |
| "I can combine some phases to save time" | Phases exist for a reason. Shortcuts compound into missed requirements. |
| "One agent can handle all implementation" | Parallel agents = faster delivery + focused context per agent. |
| "Tests can be added later" | Tests written later are worse. Each task includes tests. |
| "I'll just pass the file paths, agents can read them" | Agents start from zero context. They won't know what to look for. Copy the FULL content into their prompts. File paths are backup references, not primary context. |
| "The agent will figure out which components to use" | No. Without the explicit component table, agents will import whatever they find first. Paste the full design library catalog. |
| "Agents can figure out context from the tech spec alone" | Tech specs say WHAT to build, not WHY. Without PRD context, agents make wrong UX decisions. |
| "Agents don't need Sourcegraph, I already researched" | Agents encounter implementation-specific questions that planning didn't anticipate. Give them tools to self-serve. |
| "One round of questions is enough" | First-round answers reveal new information requiring follow-ups. Iterative deepening catches what single-pass misses. |

---

## Quick Reference: File Locations

| File | Purpose | Created In |
|------|---------|-----------|
| `docs/plans/research/design-library.md` | Component catalog | Phase 2 |
| `docs/plans/research/feature-patterns.md` | Architecture patterns | Phase 2 |
| `docs/plans/research/data-layer.md` | API surface, data models | Phase 2 |
| `docs/plans/research/codebase-context.md` | Unified context (synthesis) | Phase 2 |
| `docs/plans/research/sources.md` | PRD and design source metadata | Phase 0 |
| `docs/plans/research/gap-analysis-qa.md` | Full Q&A from deep-dive questioning | Phase 5 |
| `docs/plans/tech-spec.md` | Implementation blueprint | Phase 6 |
| `~/.claude/memory/prod-mgr-toolkit/prd-oneshotter/codebase-context.md` | Cached stable patterns | Phase 2 (updated) |
| `references/tech-spec-template.md` | Template for Phase 6 | Pre-created |
