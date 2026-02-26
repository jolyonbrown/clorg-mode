# clorg-mode

**org-mode, but you talk to it.**

A Claude Code-powered personal productivity system that replicates the power of Emacs Org mode through natural language and slash commands, backed by plain markdown files you own.

---

## Project Overview

### What This Is

clorg-mode is a task management, time tracking, note capture, and personal organisation system that runs inside Claude Code. Instead of learning Emacs keybindings, you talk to Claude. Instead of `.org` files with special syntax, you use markdown files with lightweight YAML front matter. Instead of navigating files yourself, you ask Claude to assemble *views* of your data.

### The Core Insight

In Org mode, every command is a keybinding that manipulates structured text. In clorg-mode, Claude Code manipulates structured text when you ask it in English. The files are the database. Claude is the query engine, the view renderer, and the command processor.

### Design Principles

1. **Plain text, you own it** — Everything is markdown files in a directory. No database, no proprietary format, no vendor lock-in. Works with Obsidian, VS Code, or any text editor.
2. **Talk, don't configure** — No setup wizards. Tell Claude what you want. "Add a task to check the LDAP replication logs, it's urgent, due Friday."
3. **Views, not navigation** — You never browse files. You ask for what you want to see: "What's overdue?", "Show me this week", "What's tagged NHS?". Claude reads the files and assembles a formatted view.
4. **Progressive complexity** — Start with just tasks. Add time tracking when you need it. Add projects when you need them. The system grows with you.
5. **Mobile-first capture** — Via Claude Code Remote Control, capture tasks and check status from your phone. The files live on your machine; the phone is just a window.

---

## Architecture

### Directory Structure

```
~/clorg/
├── CLAUDE.md              # System prompt — the brain of clorg-mode
├── .clorg/
│   ├── config.yaml        # User preferences and custom states
│   └── templates/         # Task/note templates
│       ├── task.md
│       ├── note.md
│       └── project.md
├── inbox/                 # Quick captures land here
│   └── *.md
├── tasks/                 # Active task files
│   └── *.md
├── projects/              # Project files (contain sub-tasks)
│   └── *.md
├── archive/               # Completed/cancelled tasks
│   └── YYYY/MM/*.md
├── notes/                 # Reference notes, meeting notes, etc.
│   └── *.md
├── logs/                  # Time tracking and daily logs
│   ├── timesheet.csv      # Machine-readable time log
│   └── daily/
│       └── YYYY-MM-DD.md  # Generated daily views (cached)
└── views/                 # Generated view outputs (ephemeral)
    └── *.md
```

### File Format: Tasks

Each task is a single markdown file. The filename is a slugified version of the title with a timestamp prefix for uniqueness.

```markdown
---
id: 20250225-143022
title: Fix HAProxy connection timeout issue
status: IN-PROGRESS
priority: high
created: 2025-02-25T14:30:22Z
due: 2025-02-28
scheduled: 2025-02-26
tags: [nhs, infrastructure, haproxy]
project: nhs-spine
estimate: 2h
clocked: 45m
blocked_by: []
---

# Fix HAProxy connection timeout issue

## Context
The connection timeouts on the Spine proxy are causing intermittent 504s during peak hours.
Need to review the `timeout server` and `timeout connect` values.

## Notes
- 2025-02-25: Checked current config, timeout server is 30s which seems low
- Current error rate is ~0.3% but spikes to 2% between 09:00-10:00

## Checklist
- [x] Pull current HAProxy config
- [ ] Compare timeout values with upstream SLA
- [ ] Test new values in staging
- [ ] Deploy to production
```

### File Format: Projects

Projects are markdown files that define a grouping. They don't contain tasks — they reference them via tags. Claude assembles the project view dynamically.

```markdown
---
id: proj-nhs-spine
title: NHS Spine Infrastructure
status: active
tags: [nhs, infrastructure]
---

# NHS Spine Infrastructure

Ongoing maintenance and optimisation of the national Spine proxy infrastructure.

## Goals
- Reduce 504 error rate below 0.1%
- Complete HAProxy upgrade to 2.9
- Migrate monitoring to new Splunk indexes
```

### File Format: Notes

```markdown
---
id: 20250225-150000
title: Meeting with NHS Digital re: Spine capacity
type: meeting-note
created: 2025-02-25T15:00:00Z
tags: [nhs, meeting]
project: nhs-spine
---

# Meeting with NHS Digital re: Spine capacity

Attendees: ...

## Key Points
...

## Action Items
- [ ] Review capacity projections → task created: 20250225-153000
```

### Configuration: `.clorg/config.yaml`

```yaml
# Task state sequences — customise to your workflow
states:
  default: [TODO, IN-PROGRESS, DONE]
  full: [TODO, NEXT, IN-PROGRESS, WAITING, BLOCKED, DONE, CANCELLED]
  
# Which states count as "open" vs "closed"
open_states: [TODO, NEXT, IN-PROGRESS, WAITING, BLOCKED]
closed_states: [DONE, CANCELLED]

# Priority levels
priorities: [critical, high, medium, low]

# Default tags (for autocomplete/consistency)
tags:
  - nhs
  - infrastructure
  - haproxy
  - splunk
  - career
  - blog
  - anthropic
  - personal

# Archive settings
archive:
  auto_archive_after_days: 7  # Auto-archive DONE tasks after 7 days
  archive_path: archive/

# Time tracking
time_tracking:
  round_to_minutes: 5
  daily_target_hours: 6

# Daily view preferences
daily_view:
  show_overdue: true
  show_scheduled: true
  show_in_progress: true
  show_blocked: true
  lookahead_days: 3
```

---

## The Navigation Model: Views, Not Files

This is the key conceptual difference from traditional file-based systems. **You never open or browse task files directly.** Instead, you ask Claude for *views* — filtered, formatted presentations of your data assembled on the fly.

### How It Works

1. You issue a command (slash command or natural language)
2. Claude reads the relevant markdown files from the filesystem
3. Claude filters, sorts, and formats the data
4. Claude presents a rendered view in the terminal (or on mobile via Remote Control)
5. If you want to act on something, you tell Claude ("mark that first one as done", "reschedule the HAProxy task to Monday")

### Why This Works

- **No context switching** — you never leave the conversation to go find a file
- **Fuzzy matching** — you can say "the HAProxy thing" and Claude knows which task you mean
- **Cross-file queries** — "show me everything tagged NHS that's overdue" scans all files instantly
- **Dynamic grouping** — "group my tasks by project" or "show me today by priority" — views are generated, not pre-defined
- **Natural follow-ups** — after seeing a view, you can immediately act: "push the Splunk task to next week and add a note saying I'm waiting on index approval"

### View Types

| View | Trigger | What It Shows |
|------|---------|---------------|
| Today | `/today` | Due today + overdue + scheduled today + in-progress items |
| Week | `/week` | This week's tasks by day, plus unscheduled high-priority items |
| Agenda | `/agenda` or `/agenda 14` | N-day lookahead with deadlines and scheduled items |
| Project | `/project nhs-spine` | All tasks/notes for a project, grouped by status |
| Overdue | `/overdue` | Everything past its due date |
| Waiting | `/waiting` | All WAITING/BLOCKED items with context on what they're waiting for |
| Inbox | `/inbox` | Unprocessed captures that need triaging |
| Timesheet | `/timesheet` or `/timesheet this week` | Time logged, by task and project |
| Search | `/search <query>` | Full-text search across all files |
| Dashboard | `/dashboard` | High-level overview: counts by status, overdue count, hours logged this week, upcoming deadlines |

---

## Slash Commands

### Task Management

| Command | Action |
|---------|--------|
| `/add <description>` | Create a new task (Claude infers priority, tags, project from context) |
| `/quick <description>` | Rapid capture to inbox — minimal metadata, triage later |
| `/done <task ref>` | Mark task as DONE, log completion time |
| `/start <task ref>` | Set status to IN-PROGRESS, start time clock |
| `/stop` | Stop current clock, log time |
| `/block <task ref> <reason>` | Set to BLOCKED with reason |
| `/wait <task ref> <reason>` | Set to WAITING with context |
| `/defer <task ref> <date>` | Reschedule task |
| `/priority <task ref> <level>` | Change priority |
| `/tag <task ref> <tags>` | Add tags |
| `/archive` | Move completed tasks older than threshold to archive |

### Views (as above)

| Command | Action |
|---------|--------|
| `/today` | Show today's view |
| `/week` | Show this week |
| `/agenda [days]` | Show agenda for N days (default: 7) |
| `/project <name>` | Show project view |
| `/overdue` | Show overdue items |
| `/waiting` | Show blocked/waiting items |
| `/inbox` | Show inbox items for triage |
| `/timesheet [period]` | Show time tracking summary |
| `/search <query>` | Search across all files |
| `/dashboard` | High-level overview |

### System

| Command | Action |
|---------|--------|
| `/init` | Initialise a new clorg directory |
| `/review` | Weekly review — walks through overdue, stale, and unscheduled items interactively |
| `/triage` | Process inbox items one by one — set priority, tags, dates |
| `/cleanup` | Archive old completed tasks, flag stale in-progress items |

---

## Natural Language Interface

Beyond slash commands, Claude should understand and respond to natural language. Examples:

### Adding Tasks
- "I need to check the LDAP replication logs by Friday, it's high priority"
- "Add a task to write the blog post about clorg-mode"
- "Remind me to review the Splunk license usage next Tuesday"

### Querying
- "What am I working on right now?"
- "What's due this week?"
- "Show me everything related to the Spine project"
- "How much time did I spend on NHS stuff this week?"
- "What's blocking me?"
- "What did I get done yesterday?"

### Modifying
- "The HAProxy task is blocked — waiting on the firewall change"
- "Push the blog post to next week"
- "Actually make that critical priority"
- "I just finished the Splunk query optimisation"
- "Clock 45 minutes against the HAProxy task"

### Context-Aware Interaction
- After viewing today's tasks: "Start the first one"
- After viewing a project: "What's the most urgent thing here?"
- "I've got 2 hours, what should I focus on?"
- "I'm on my phone on the bus, anything I can do from here?"

---

## Time Tracking

### How It Works

Time tracking is opt-in per task. When you `/start` a task, Claude notes the timestamp. When you `/stop` or `/start` a different task, it calculates elapsed time and updates both the task file and the central timesheet.

### Task File Update
```markdown
---
clocked: 2h15m
clock_log:
  - start: 2025-02-25T09:15:00Z
    end: 2025-02-25T10:30:00Z
    duration: 1h15m
  - start: 2025-02-25T14:00:00Z
    end: 2025-02-25T15:00:00Z
    duration: 1h
---
```

### Timesheet CSV
```csv
date,task_id,task_title,project,start,end,duration_minutes,tags
2025-02-25,20250225-143022,Fix HAProxy timeout,nhs-spine,09:15,10:30,75,nhs;infrastructure;haproxy
2025-02-25,20250225-143022,Fix HAProxy timeout,nhs-spine,14:00,15:00,60,nhs;infrastructure;haproxy
```

### Manual Time Entry
- "Clock 45 minutes against the HAProxy task"
- "I spent 2 hours on Splunk queries this morning"
- Claude updates both the task and the timesheet

---

## Capture Flow (Mobile + Desktop)

### Desktop (Terminal)
```
> /quick Check if Spine auth errors correlate with the LDAP cert expiry

Created: inbox/20250225-161500-check-spine-auth-errors.md
```

### Mobile (via Remote Control)
Same commands work. The key insight: when you're on your phone, you're probably capturing or checking, not doing deep task management. So the mobile experience should be optimised for:

1. **Quick capture** — voice-to-text → `/quick <whatever you said>`
2. **Status check** — `/today` to see what's on
3. **Quick updates** — "mark the Splunk task as done" / "I'm blocked on the firewall change"
4. **Triage inbox** — `/triage` to process captures from the day

### Inbox Processing
The inbox is a holding pen. Items captured via `/quick` have minimal metadata. The `/triage` command walks through them:

```
INBOX: "Check if Spine auth errors correlate with the LDAP cert expiry"
  → What project? (nhs-spine)
  → Priority? (high)
  → Due date? (this Friday)
  → Tags? (nhs, ldap, spine)
  → Created task: tasks/20250225-161500-check-spine-auth-errors.md
  
Next item...
```

Claude drives this interactively. On mobile, you can just respond with quick answers.

---

## Weekly Review: `/review`

Inspired by GTD and Org mode's agenda review. Claude walks you through:

1. **Overdue items** — For each: reschedule, complete, or cancel?
2. **Stale in-progress** — Started more than 3 days ago with no updates. Still working on it? Blocked?
3. **Waiting/Blocked** — Any updates? Can we unblock anything?
4. **Completed this week** — Summary of what you got done (morale boost)
5. **Time summary** — Hours logged by project this week vs target
6. **Upcoming deadlines** — Next 7 days, anything need prep?
7. **Unscheduled high-priority** — Should these get dates?
8. **Inbox check** — Anything left untriaged?

This should take 10-15 minutes and leaves you with a clean, current system.

---

## CLAUDE.md — The System Prompt

This is the most critical file. It tells Claude Code how to behave as clorg-mode. It should be placed at `~/clorg/CLAUDE.md`.

### What CLAUDE.md Must Define

1. **Identity** — "You are clorg-mode, a task management system. When the user opens a session in this directory, you are their productivity assistant."

2. **File format specs** — Exact YAML front matter fields, valid states, how to generate IDs (timestamp-based: `YYYYMMDD-HHMMSS`)

3. **Slash command definitions** — What each command does, which files to read/write

4. **View rendering rules** — How to format each view type for terminal output. Use unicode box-drawing characters, colour via ANSI codes where supported, and clean alignment. Keep views scannable — this is critical for mobile.

5. **Natural language understanding** — Guidelines for inferring priority, tags, project from context. E.g., if the user mentions "HAProxy" or "Spine", auto-tag with `nhs` and `infrastructure`.

6. **State machine rules** — Valid state transitions (e.g., can't go from DONE back to TODO without explicit confirmation)

7. **Time tracking rules** — How to handle clock start/stop, what to do if user starts a new task without stopping the previous one (auto-stop the previous)

8. **Archive rules** — When to suggest archiving, how to move files

9. **Context awareness** — Claude should remember what view was last shown and allow references like "the first one", "that HAProxy task", "the third item"

10. **Error handling** — What to do when a task reference is ambiguous (ask for clarification), when files are malformed (fix and report), when conflicts arise

---

## Build Plan

### Phase 1: Foundation (MVP)
**Goal: Working task management with slash commands**

1. Create the directory structure (`/init`)
2. Write `CLAUDE.md` with core system prompt
3. Create `.clorg/config.yaml` with defaults
4. Create file templates
5. Implement task CRUD:
   - `/add` — create task file with YAML front matter
   - `/quick` — create inbox item
   - `/done` — update status, move to archive after threshold
   - `/start`, `/stop` — status changes (no time tracking yet)
6. Implement core views:
   - `/today` — due today + overdue + in-progress
   - `/inbox` — show unprocessed items
   - `/search` — grep across files
7. Test with real tasks

### Phase 2: Views & Navigation
**Goal: Rich querying and the full view system**

1. Implement all view commands:
   - `/week`, `/agenda`, `/project`, `/overdue`, `/waiting`, `/dashboard`
2. Implement natural language queries
3. Add context-awareness (remember last view, allow "the first one" references)
4. Polish view formatting for terminal readability
5. Test views on mobile via Remote Control

### Phase 3: Time Tracking
**Goal: Full time tracking with reporting**

1. Implement clock start/stop in task files
2. Create and maintain `timesheet.csv`
3. Implement `/timesheet` views with period filtering
4. Add manual time entry via natural language
5. Add time estimates and tracking against estimates

### Phase 4: Review & Workflow
**Goal: GTD-style review and triage workflows**

1. Implement `/review` — the weekly review walk-through
2. Implement `/triage` — interactive inbox processing
3. Implement `/cleanup` — automated archiving and staleness detection
4. Add smart suggestions ("you haven't touched this in 5 days, still active?")

### Phase 5: Advanced Features
**Goal: Power features for long-term use**

1. Recurring tasks (daily, weekly, monthly)
2. Task dependencies and blocking chains
3. Project-level progress tracking and reporting
4. Daily log generation (what got done today, auto-generated)
5. Export/reporting (weekly summary for blog, timesheet for invoicing)
6. Integration with git (commit task state changes)

---

## Technical Notes for Implementation

### ID Generation
Use timestamp-based IDs: `YYYYMMDD-HHMMSS`. These sort chronologically and are human-readable. For the rare case of collision (two tasks in the same second), append a counter: `20250225-143022-2`.

### File Naming
Slugify the title and prepend the ID: `20250225-143022-fix-haproxy-timeout.md`. Truncate slug at 50 chars.

### YAML Parsing
Claude Code should read YAML front matter between `---` delimiters. Use a simple parser — don't pull in heavy dependencies. The front matter is intentionally simple enough to parse with basic string operations if needed.

### View Rendering
Views should be formatted as clean, scannable terminal output. Example for `/today`:

```
═══════════════════════════════════════════════════
  📅 TODAY — Wednesday 26 February 2025
═══════════════════════════════════════════════════

  🔴 OVERDUE
  ──────────
  [!] Fix Spine auth error logging          due: Mon 24   #nhs #spine
  [!] Update Splunk index retention         due: Tue 25   #nhs #splunk

  🟡 DUE TODAY
  ──────────
  [ ] Review HAProxy timeout config         pri: high     #nhs #haproxy
  [ ] Write clorg-mode blog post            pri: medium   #blog

  🔵 IN PROGRESS
  ──────────
  [~] Migrate Splunk dashboards             started: 2d ago  ⏱ 3h15m
  
  🟢 SCHEDULED TODAY
  ──────────
  [ ] Weekly team sync notes                pri: medium   #nhs
  
  📬 INBOX (3 items) — run /triage to process

═══════════════════════════════════════════════════
```

### Mobile Considerations
When rendering views for mobile (narrower screen), Claude should:
- Use shorter formats (abbreviate tags, drop less critical columns)
- Keep the most important info (title, due date, status) always visible
- Prefer vertical layout over horizontal density

### Conflict Handling
Since files live on the local filesystem, conflicts are unlikely (single user). However, if Claude detects a file has changed between read and write (e.g., user edited it in VS Code), it should warn before overwriting.

---

## Success Criteria

The system is working when:

1. Jolyon can open a Claude Code session in `~/clorg/`, type `/today`, and see a useful view of what needs doing
2. He can add tasks in natural language without thinking about file format
3. He can check and update tasks from his phone via Remote Control
4. The weekly review actually happens because `/review` makes it low-friction
5. After 2 weeks, the system has real data and is genuinely useful — not abandoned like previous attempts
6. The README makes Emacs users curious and everyone else excited

---

## README.md (Draft)

```markdown
# clorg-mode

**org-mode, but you talk to it.**

A personal productivity system powered by Claude Code. Plain markdown files.
Natural language interface. No Emacs required.

## What is this?

clorg-mode gives you the power of Emacs Org mode — task management, time tracking,
agenda views, capture, and review — through conversation with Claude Code.

Instead of learning keybindings, you talk. Instead of `.org` syntax, you use markdown.
Instead of navigating files, you ask for views.

## Quick Start

\```bash
cd ~/clorg
claude
> /init
> /add Write the quarterly report, due Friday, high priority
> /today
\```

## Requirements

- Claude Code CLI (Pro or Max subscription)
- That's it. That's the list.

## Mobile

With Claude Code Remote Control, start a session on your desktop and continue
from your phone:

\```bash
claude remote-control
\```

Scan the QR code and manage your tasks from anywhere.

## Philosophy

Your tasks are plain markdown files. No database. No proprietary format.
No vendor lock-in. If you stop using clorg-mode tomorrow, you still have
a folder of perfectly readable markdown files.

## Name

It's "Claude" + "Org mode". Yes, we're very pleased with ourselves.
```
