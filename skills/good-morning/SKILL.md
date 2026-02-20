---
name: good-morning
description: >
  Daily morning workflow automation. Syncs meeting notes, calendar, email, and
  task board, then walks Matt through each section interactively so nothing
  gets lost in a wall of text. Invoke with /good-morning at the start of the day.
---

# Good Morning Skill

Run this skill exactly as written. Do not skip phases or reorder them.

**Critical rule**: After presenting each step, **STOP and wait for user input** before continuing to the next step. The user may react, ask questions, or say "next" to continue.

## Quick Reference: Tool Parameters

| Service | Tool prefix | Key params |
|---------|------------|------------|
| Granola | `mcplocker:granola` | `time_range`: this_week, last_week, last_30_days |
| Gmail | `mcplocker:google_gmail` | Gmail search operators, `label_ids: ["INBOX"]` |
| Calendar | `mcplocker:google_calendar` | `time_min`/`time_max` ISO 8601 |
| Linear | `linear` | `assignee="me"`, `team="Product Managers"` |

## Phase 0: Context Setup

Before dispatching subagents, do this in the main agent:

1. **Read memory**: If the memory file at `~/.claude/memory/prod-mgr-toolkit/good-morning.md` does not exist, create it by copying the template from `memory.md` in this skill's directory. Then read the memory file.
2. **Determine day context**:
   - Get today's day of week
   - **Monday**: Granola `time_range=last_week`, Gmail query `newer_than:3d`, flag "weekly review mode"
   - **Other days**: Granola `time_range=this_week`, Gmail query `newer_than:1d`
3. **Extract previous action items** from memory's "Active Action Items" section (needed for Step 4)

Store these values for subagent prompts.

## Phases 1-4: Parallel Data Gathering

Dispatch exactly 4 subagents using the `Task` tool with `subagent_type: "general-purpose"`. All 4 MUST be dispatched in a single message (parallel execution). Pass each subagent the specific parameters determined in Phase 0.

### Subagent 1: Yesterday's Meetings (Granola)

```
Prompt the subagent with:
- Use ToolSearch to find and load granola tools
- Call granola list_meetings with time_range=[value from Phase 0]
- Filter to yesterday's meetings only (or Friday + weekend if Monday)
- For each meeting, call get_meetings to get full details
- CONCISENESS RULE: Return 2-3 sentence summary per meeting + bullet-point action items only. No full transcripts, no lengthy discussion recaps.
- Return per-meeting: title, attendees list, concise summary, action items
- Format as structured markdown
```

### Subagent 2: Today's Calendar (Google Calendar)

```
Prompt the subagent with:
- Use ToolSearch to find and load google_calendar tools
- Call get_events with time_min=start of today (ISO 8601), time_max=end of today (ISO 8601)
- Return: full schedule with times, titles, attendee lists, descriptions
- Flag meetings with 3+ attendees or "review"/"standup"/"planning" in title
- Format as structured markdown
```

### Subagent 3: Important Emails (Gmail)

```
Prompt the subagent with:
- Use ToolSearch to find and load google_gmail tools
- Call list_threads with query="is:unread [day context query from Phase 0]", label_ids=["INBOX"]
- For top 10 most recent unread threads, call get_thread for full context
- CONCISENESS RULE: Only return actionable threads — skip calendar updates, automated notifications, newsletters, and marketing emails. An "actionable" thread is one that requires a response, decision, or follow-up.
- Return: total unread count, list of actionable threads with sender, subject, snippet, thread_id
- Format as structured markdown
```

### Subagent 4: Linear Task Board

```
Prompt the subagent with:
- Use ToolSearch to find and load linear tools
- Call list_issues with assignee="me", team="Product Managers", includeArchived=false
- Call list_issue_statuses with team="Product Managers"
- Filter out terminal states (Done, Cancelled, Duplicate)
- Group remaining by status: Blocked → In Progress → In Review → Todo → Backlog
- CONCISENESS RULE: Per issue return only: identifier, title, priority, due date, labels. No full descriptions.
- Also return: issues completed (Done) in last 7 days for dedup reference
- Format as structured markdown
```

## Interactive Presentation (Steps 1-6)

After all subagents return, present results **one step at a time**. After each step, STOP and wait for user input before continuing.

---

### Step 1: Yesterday's Meetings + Action Item Triage

Using both the Granola subagent (Subagent 1) and Linear subagent (Subagent 4) results, present meetings with annotated action items:

**A) Per meeting:**
- 2-3 sentence summary
- Bullet-point action items, each annotated:
  - **"Already tracked: PM-XX"** — if it fuzzy-matches an active Linear task
  - **"Previously completed"** — if it matches a Done/Cancelled task from last 7 days
  - **"New"** — if no Linear match found
- Check memory.md Rejected Suggestions — don't surface rejected items as "New"

**B) Ask which new items to add to Linear:**
- List only the "New" (untracked) items with numbered checkmark-style formatting
- **End with**: "Which of these should I add to Linear? (list numbers, 'all', or 'none') — then we'll move to today's calendar."

**STOP. Wait for user input.**

**C) After user responds, create Linear tasks:**
- For each confirmed item, use `linear:create_issue` with:
  - Title from the action item
  - Team: "Product Managers"
  - Assignee: "me"
  - Status: Todo (or as user specifies)
  - Description referencing the source meeting
- Confirm created tasks with their identifiers (e.g., "Created PM-71, PM-72")
- Then proceed to Step 2

---

### Step 2: Today's Schedule

Present the Calendar subagent results:
- Time, title, and attendees for each meeting
- Flag notable meetings (large groups, reviews, planning sessions)

**End with**: "Ready for email?"

**STOP. Wait for user input.**

---

### Step 3: Email Highlights

Present the Gmail subagent results:
- Only actionable threads (skip noise)
- Sender, subject, and 1-line summary per thread
- Total unread count as context

**End with**: "Ready for your task board?"

**STOP. Wait for user input.**

---

### Step 4: Interactive Board Review

Start with a one-line overview: total active issue count across all statuses (e.g., "You have **14 active issues** across your board.").

**A) Per-ticket review by status group**

Walk through each non-empty status group in this order: **Blocked → In Progress → In Review → Todo → Backlog**. Skip any group that has zero issues.

For each group:
1. Print a status group header (e.g., "**Blocked (2 issues)**")
2. For each ticket in the group, present a one-line summary:
   - Format: `PM-XX: Title — Priority, due DATE` (omit due date if none)
3. Immediately follow that ticket with an `AskUserQuestion` prompt:
   - Options:
     - **"No changes"** — move to the next ticket
     - **"Change status"** — follow up by asking which status to move to, then call `linear:update_issue` to apply
     - **"Change priority"** — follow up by asking the new priority, then call `linear:update_issue` to apply
   - (The user can always select "Other" for custom actions like adding a comment, updating the title, reassigning, etc.)
4. Process the user's response fully before proceeding to the next ticket
5. After all tickets in a group are reviewed, move to the next non-empty group

**B) Deadlines** — extracted from the board (present after all tickets are reviewed):
- Overdue (past due date, still open)
- Due Today
- Due This Week

**C) Action Item Follow-Up** — using the previous action items from Phase 0 memory:
- Linear task updated since last session? → "PM-45: Moved from Todo → In Progress"
- Still unchanged? → "PM-102: No progress — still In Progress"
- Skip this sub-section entirely if there are no previous action items in memory

**Board health flags** (only if notable):
- Issues In Progress >2 weeks
- Large Todo backlog without due dates
- Blocked items with no recent activity

**End with**: "Ready for priorities?"

**STOP. Wait for user input.**

---

### Step 5: Suggested Priorities

Perform cross-source linking silently, then present the results as **annotated priorities**:

**Cross-source linking** (do this analysis, don't show it as a separate section):
- Match email threads → Linear tasks (look for issue IDs like `PM-XX` in subjects/bodies)
- Match email senders → today's meeting attendees

**Present the output as**:

1. **Top 3 suggested priorities** — cross-referenced and annotated. E.g.:
   - "PM-40 (digital advice doc) — Jackie asked for this by Friday (email + yesterday's meeting)"
   - "Reply to Alex's pricing thread — ties to PM-67 (blocked, needs your input)"
   - "Prep for 2pm design review — related to PM-102 (in progress)"

2. **New untracked email items** — genuinely new items from email that don't match any Linear task (meeting items were already handled in Step 1)

3. **Memory-informed suggestions** (use learned patterns from memory.md):
   - Reference Priority Signals, Time Patterns, Task Type Preferences
   - E.g., "You usually tackle client-facing work first" → surface those higher
   - "On Mondays you prefer planning" → suggest planning tasks on Mondays

**End with**: "What would you like to focus on today?"

**STOP. Wait for user input.**

---

### Step 6: Learn from Decisions

After the user responds with their priorities:

1. **Update `~/.claude/memory/prod-mgr-toolkit/good-morning.md`**:
   - **Recent Decisions**: Add today's choice (keep last 5, prune oldest)
   - **Active Action Items**: Replace entirely with today's commitments
   - **Rejected Suggestions**: Add any declined meeting-sourced tasks
   - **Promote patterns**: If a pattern appears 3+ times in Recent Decisions, move it to the permanent section (Priority Signals, Time Patterns, etc.)
2. **Ask at most 1 clarifying question** if reasoning is unclear
3. Confirm the day's plan is set

## Common Mistakes

- **Don't skip Phase 0**: Memory and day context must be read before subagents launch
- **Don't serialize subagents**: All 4 must dispatch in one message for parallel execution
- **Don't re-suggest rejected items**: Always check Rejected Suggestions before surfacing
- **Don't auto-create Linear tasks**: In Step 1, present new meeting items and let Matt choose which to add. Never create without confirmation.
- **Don't update memory before user responds**: Step 6 happens after Step 5 interaction
- **Don't store sensitive content in memory**: Only patterns, preferences, and task IDs
- **Don't dump everything at once**: Each step must STOP and wait for user input
- **Monday handling**: Use `last_week` for Granola and `newer_than:3d` for Gmail

## Monday / Weekly Review Extras

When it's Monday:
- Cover Friday + weekend in meeting/email review
- Add a "Last Week Recap" section summarizing completed work
- Flag any items that were In Progress all last week without moving
- Suggest weekly planning priorities
