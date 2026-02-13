# prod-mgr-toolkit

Product management skills for Claude Code. Five interactive workflows covering the full PM lifecycle: daily standup, PRD drafting, PRD enhancement, feature implementation, and task execution.

## Skills

| Skill | Trigger Phrases | Description |
|-------|----------------|-------------|
| **good-morning** | `/good-morning` | Daily morning workflow: syncs Granola meetings, Google Calendar, Gmail, and Linear tasks. Walks through each section interactively with cross-source priority suggestions. |
| **prd-drafter** | `draft a PRD`, `write a PRD`, `PRD for this idea` | 7-phase interactive PRD drafting from any starting point (sentence, pitch, research notes). Problem-first exploration before solutions. Runs in plan mode. |
| **prd-enhance** | `enhance this PRD`, `review this PRD`, `strengthen this spec` | 10-phase PRD enhancement: competitive scan, expert review (Product/Eng/Design), Dovetail research, Preset analytics, spec detailing with traceability matrix. |
| **prd-oneshotter** | `build this feature`, `implement this PRD`, `one-shot this feature` | 8-phase feature implementation: PRD ingestion, codebase research, design decomposition, deep-dive questioning, tech spec generation, and parallel agent team coding. |
| **task-crusher** | `crush tasks`, `work through my tasks`, `what should I work on` | Interactive Linear task execution: loads your board, lets you pick tasks, guides research/writing/review work, updates Linear on completion. |

## Installation

### From GitHub

```bash
claude plugin install github.com/mebreuer/prod-mgr-toolkit
```

### From Wealthsimple Marketplace

The plugin is available in the WS Claude Marketplace. It will be installed automatically if you have the marketplace configured.

## Memory

On first use, skills that track state will create memory files at:

```
~/.claude/memory/prod-mgr-toolkit/
├── good-morning.md        # Daily patterns, priority signals, recent decisions
├── task-crusher.md         # Work preferences, session history, task notes
└── prd-oneshotter/
    └── codebase-context.md # Cached codebase patterns (design library, conventions)
```

These files persist across sessions. Template files in each skill's directory are used to initialize them.

## Dependencies

These skills integrate with various MCP servers. All are optional — skills gracefully degrade when tools are unavailable.

| MCP Server | Used By | Purpose |
|-----------|---------|---------|
| Granola | good-morning | Meeting notes and transcripts |
| Google Calendar | good-morning | Today's schedule |
| Google Gmail | good-morning | Unread email triage |
| Linear | good-morning, task-crusher | Task board, issue management |
| Dovetail | prd-drafter, prd-enhance | User research insights |
| Preset | prd-drafter, prd-enhance | Analytics and usage data |
| Notion | prd-enhance, prd-oneshotter | PRD source and push-back |
| Figma | prd-oneshotter | Design decomposition |
| Sourcegraph | prd-oneshotter | Codebase research |

## Structure

```
prod-mgr-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── good-morning/
│   │   ├── SKILL.md
│   │   └── memory.md                (template)
│   ├── prd-drafter/
│   │   ├── SKILL.md
│   │   ├── prd-drafter-patterns.md
│   │   └── references/
│   │       └── prd-draft-template.md
│   ├── prd-enhance/
│   │   ├── SKILL.md
│   │   ├── prd-patterns.md
│   │   └── references/
│   │       ├── expert-review-checklist.md
│   │       ├── prd-output-template.md
│   │       └── preset-query-patterns.md
│   ├── prd-oneshotter/
│   │   ├── SKILL.md
│   │   ├── memory/
│   │   │   └── .gitkeep
│   │   └── references/
│   │       └── tech-spec-template.md
│   └── task-crusher/
│       ├── SKILL.md
│       └── memory.md                (template)
├── README.md
└── LICENSE
```

## License

MIT
