# clorg-mode

**org-mode, but you talk to it.**

A personal productivity system powered by [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Task management, time tracking, agenda views, capture, weekly review — all through natural language and slash commands, backed by plain markdown files you own.

```
> I need to fix the HAProxy timeout issue by Friday, it's urgent

  ✓ Created: Fix the HAProxy timeout issue
    priority: critical  ·  due: Fri 28 Feb  ·  #haproxy
```

## Quick Start

```bash
# 1. Clone and install
git clone https://github.com/jolyonbrown/clorg-mode.git
mkdir -p ~/clorg
cp -r clorg-mode/dist/. ~/clorg/

# 2. Open Claude Code in your clorg directory
cd ~/clorg
claude

# 3. Initialise and go
> /init
> /add Write the quarterly report, due Friday, high priority
> /today
```

## What It Does

```
> /today

═══════════════════════════════════════════════════
  📅 TODAY — Thursday 26 February 2026
═══════════════════════════════════════════════════

  🔴 OVERDUE
  ──────────
  1. [!] Fix Spine auth error logging     due: Tue 24 (2d late)  #nhs

  🟡 DUE TODAY
  ──────────
  2. [ ] Review HAProxy timeout config    pri: high     #haproxy

  🔵 IN PROGRESS
  ──────────
  3. [~] Migrate Splunk dashboards        ⏱ 3h15m  🔴 clocking

  📬 INBOX (3 items) — run /triage to process

═══════════════════════════════════════════════════
```

You never open files. You ask for views, and Claude assembles them. You talk to it naturally, and it manages the files for you.

## Commands

### Task Management

| Command | What it does |
|---------|-------------|
| `/add <desc>` | Create a task (infers priority, due date, tags, project) |
| `/quick <desc>` | Fast capture to inbox — triage later |
| `/done <ref>` | Mark task as done |
| `/start <ref>` | Start working (sets in-progress, starts clock) |
| `/stop` | Stop current task and clock |
| `/block <ref> <reason>` | Mark as blocked |
| `/wait <ref> <reason>` | Mark as waiting |
| `/defer <ref> <date>` | Reschedule |
| `/priority <ref> <level>` | Change priority |
| `/tag <ref> <tags>` | Add tags |
| `/recur <ref> <pattern>` | Make it recurring (daily, weekly, monthly) |

### Views

| Command | What it shows |
|---------|--------------|
| `/today` | Due today + overdue + in-progress + scheduled |
| `/week` | This week by day + unscheduled high-priority |
| `/agenda [N]` | N-day lookahead (default: 7) |
| `/project <name>` | Project view with progress stats |
| `/overdue` | Everything past due |
| `/waiting` | All blocked/waiting items with reasons |
| `/dashboard` | High-level overview: counts, deadlines, time |
| `/inbox` | Unprocessed captures |
| `/search <query>` | Full-text search across all files |
| `/timesheet [period]` | Time tracking summary with progress bars |
| `/log [date]` | Daily log of activity |

### Workflow

| Command | What it does |
|---------|-------------|
| `/review` | Weekly review — walks through everything interactively |
| `/triage` | Process inbox items one by one |
| `/cleanup` | Archive done tasks, flag stale items |
| `/export summary` | Generate a status report |

### Natural Language

You don't need slash commands. Just talk:

- *"I need to check the LDAP logs by Friday, it's urgent"*
- *"What's blocking me?"*
- *"The HAProxy task is done"*
- *"Push the blog post to next week"*
- *"How much time did I spend on NHS stuff this week?"*
- *"Clock 45 minutes against the Splunk task"*
- *"Start the first one"* (after viewing a list)

## How It Works

Everything is **plain markdown files** with YAML front matter:

```markdown
---
id: "20250226-143022"
title: "Fix HAProxy connection timeout issue"
status: IN-PROGRESS
priority: high
due: "2025-02-28"
tags: [nhs, infrastructure, haproxy]
project: nhs-spine
clocked: "2h15m"
---

# Fix HAProxy connection timeout issue

## Notes
- 2025-02-25: Checked current config, timeout server is 30s
```

Claude reads these files, applies your commands, and writes the changes back. No database, no API, no server. Just files.

```
~/clorg/
├── CLAUDE.md         # System prompt (the brain)
├── tasks/            # One file per task
├── inbox/            # Quick captures
├── projects/         # Project definitions
├── archive/          # Completed tasks
├── notes/            # Reference material
└── logs/             # Timesheets and daily logs
```

## Mobile

Works on your phone via [Claude Code Remote Control](https://docs.anthropic.com/en/docs/claude-code):

```bash
claude --remote
```

Optimised for quick capture and status checks from anywhere.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (Pro or Max subscription)
- That's it. That's the list.

## Philosophy

Your tasks are plain markdown files. No database. No proprietary format. No vendor lock-in. If you stop using clorg-mode tomorrow, you still have a folder of perfectly readable markdown files.

This isn't a todo app. It's a **personal operating system** that grows with you — from a simple task list to a full GTD-style productivity system with time tracking, recurring tasks, dependencies, and weekly reviews.

## Learn More

See the full [Tutorial](TUTORIAL.md) for a guided walkthrough of every feature.

## Name

It's "Claude" + "Org mode". Yes, we're very pleased with ourselves.

## License

MIT
