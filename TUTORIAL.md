# clorg-mode Tutorial

A guided walkthrough of clorg-mode, from first task to full workflow.

## Setup

```bash
git clone https://github.com/jolyonbrown/clorg-mode.git
mkdir -p ~/clorg
cp -r clorg-mode/dist/. ~/clorg/
cd ~/clorg
claude
```

Once Claude opens, initialise the directory structure:

```
> /init
```

This creates all the folders, default config, and templates. You're ready to go.

## Part 1: Your First Tasks

### Adding tasks

Just tell Claude what you need to do:

```
> I need to fix the HAProxy connection timeout issue by Friday, it's urgent

  ✓ Created: Fix the HAProxy connection timeout issue
    priority: critical  ·  due: Fri 28  ·  #haproxy
```

Claude infers priority from "urgent" (→ critical), the due date from "by Friday", and tags from keywords. You can also use the slash command directly:

```
> /add Review the Splunk license usage, medium priority, due next Tuesday

  ✓ Created: Review the Splunk license usage
    priority: medium  ·  due: Tue 4 Mar  ·  #splunk
```

### Quick capture

When you just want to jot something down without thinking about metadata:

```
> /quick Check if Spine auth errors correlate with LDAP cert expiry

  📬 Captured: Check if Spine auth errors correlate with LDAP cert expiry
```

This lands in your inbox for later triage.

### Viewing your tasks

```
> /today

═══════════════════════════════════════════════════
  📅 TODAY — Thursday 26 February 2026
═══════════════════════════════════════════════════

  🟡 DUE TODAY
  ──────────
  1. [ ] Review the Splunk license usage    pri: medium  #splunk

  📬 INBOX (1 item) — run /triage to process

═══════════════════════════════════════════════════
```

Every item is numbered. You can refer back to them:

```
> start the first one

  ✓ Started: Review the Splunk license usage. Clock running.
```

## Part 2: Working With Tasks

### Status changes

```
> /start the HAProxy task

  ✓ Started: Fix the HAProxy connection timeout issue. Clock running.
    (Auto-stopped clock on: Review the Splunk license usage — logged 25m)
```

Starting a new task automatically stops the clock on the previous one.

```
> /done the HAProxy task

  ✓ Done: Fix the HAProxy connection timeout issue (1h15m clocked)
```

### Blocking and waiting

```
> the Splunk task is blocked, waiting on index approval from the platform team

  ✓ Blocked: Review the Splunk license usage
    Reason: waiting on index approval from the platform team
```

```
> /waiting

═══════════════════════════════════════════════════
  ⏳ WAITING & BLOCKED — 1 item
═══════════════════════════════════════════════════

  BLOCKED
  ──────────
  1. [🚫] Review the Splunk license usage    #splunk
     └─ waiting on index approval from the platform team    since: just now

═══════════════════════════════════════════════════
```

### Rescheduling

```
> push the Splunk task to next Friday

  ✓ Deferred: Review the Splunk license usage
    was: Tue 4 Mar → now: Fri 7 Mar
```

## Part 3: Views

### The week at a glance

```
> /week

═══════════════════════════════════════════════════
  📅 WEEK — 24 Feb – 2 Mar 2026
═══════════════════════════════════════════════════

  Friday 28
  ──────────
  1. [ ] Fix HAProxy connection timeout     pri: critical  #haproxy

  ⚡ UNSCHEDULED HIGH PRIORITY
  ──────────
  (none)

═══════════════════════════════════════════════════
```

### Looking ahead

```
> /agenda 14

═══════════════════════════════════════════════════
  📅 AGENDA — 14 days (26 Feb – 11 Mar 2026)
═══════════════════════════════════════════════════

  Fri 28 Feb
  ──────────
  1. [ ] Fix HAProxy connection timeout     pri: critical  #haproxy

  Fri 7 Mar
  ──────────
  2. [🚫] Review the Splunk license usage   pri: medium    #splunk

═══════════════════════════════════════════════════
```

### Dashboard

```
> how am I doing?

═══════════════════════════════════════════════════
  📊 DASHBOARD
═══════════════════════════════════════════════════

  Status            Count
  ─────────────────────────
  TODO                1
  BLOCKED             1
  ─────────────────────────
  Total open          2

  🔴 Overdue:  0
  📬 Inbox:    1

  📅 Upcoming Deadlines (7 days)
  ──────────
  Fri 28  Fix HAProxy connection timeout    pri: critical

═══════════════════════════════════════════════════
```

## Part 4: Time Tracking

### Automatic (clock-based)

When you `/start` a task, the clock starts. When you `/stop`, `/done`, or start a different task, the elapsed time is logged automatically.

```
> /start the HAProxy task

  ✓ Started: Fix the HAProxy connection timeout issue. Clock running.
```

*...work for a while...*

```
> /stop

  ✓ Stopped: Fix the HAProxy connection timeout issue. Logged 1h15m.
```

### Manual

```
> clock 45 minutes against the Splunk task

  ✓ Logged 45m against: Review the Splunk license usage. Total: 45m.
```

```
> I spent 2 hours on HAProxy stuff this morning

  ✓ Logged 2h against: Fix the HAProxy connection timeout issue. Total: 3h15m.
```

### Viewing your time

```
> /timesheet this week

═══════════════════════════════════════════════════
  ⏱ TIMESHEET — This week (24 Feb – 2 Mar 2026)
═══════════════════════════════════════════════════

  By Project              Hours
  ─────────────────────────────
  (no project)            4h
  ─────────────────────────────
  Total                   4h

  By Day
  ──────────
  Thu 26    4h      ████████████░░  (target: 6h)

  Detail
  ──────────
  1. Fix HAProxy connection timeout     3h15m   #haproxy
  2. Review Splunk license usage        0h45m   #splunk

═══════════════════════════════════════════════════
```

## Part 5: Projects

### Creating a project

```
> /add a project for NHS Spine Infrastructure, tags nhs and infrastructure
```

This creates a file in `projects/` that groups related tasks.

### Viewing a project

```
> /project nhs-spine

═══════════════════════════════════════════════════
  📁 PROJECT — NHS Spine Infrastructure
═══════════════════════════════════════════════════

  📊 Progress
  ──────────
  Tasks:  1/4 done (25%)  ███░░░░░░░
  Time:   4h logged

  📋 TODO
  ──────────
  1. [ ] Fix HAProxy connection timeout     pri: critical  #haproxy

  ⏳ WAITING / BLOCKED
  ──────────
  2. [🚫] Review Splunk license usage       #splunk
     └─ waiting on index approval

  ✅ DONE
  ──────────
  3. [✓] Check LDAP replication logs        ⏱ 1h15m

═══════════════════════════════════════════════════
```

## Part 6: Inbox and Triage

Quick captures land in your inbox with no metadata. Process them when you're ready:

```
> /triage

═══════════════════════════════════════════════════
  📬 TRIAGE — 1 item to process
═══════════════════════════════════════════════════

  "Check if Spine auth errors correlate with LDAP cert expiry"

  Project?  (suggest: nhs-spine)
  Priority? (suggest: medium)
  Due date? (suggest: none)
  Tags?     (suggest: nhs, spine, ldap)
```

```
> nhs-spine, high, this Friday, nhs spine ldap

  ✓ Created task: Check if Spine auth errors correlate with LDAP cert expiry
    project: nhs-spine · pri: high · due: Fri 28 · #nhs #spine #ldap

  ── TRIAGE COMPLETE ─────────────────────────────
  1 item triaged, 0 skipped, 0 deleted
```

## Part 7: Recurring Tasks

```
> make the LDAP check a weekly task

  ✓ Set to recur weekly (fixed schedule).
    When completed, next instance will be due the following Friday.
```

When you complete a recurring task, clorg-mode automatically creates the next instance:

```
> /done the LDAP check

  ✓ Done: Check LDAP replication logs (45m clocked)
  🔄 Recurring: next instance created, due Fri 7 Mar
```

## Part 8: Dependencies

Tasks can depend on other tasks:

```
> the auth error task depends on the HAProxy upgrade

  ✓ Dependency set: Check Spine auth errors → blocked by: Upgrade HAProxy to 2.9
```

When you complete a blocker, clorg-mode tells you what it unblocks:

```
> /done the HAProxy upgrade

  ✓ Done: Upgrade HAProxy to 2.9
  🔓 This unblocks: Check Spine auth errors. Set to TODO?
```

View the full chain:

```
> /deps the auth error task

═══════════════════════════════════════════════════
  🔗 DEPENDENCIES — Check Spine auth errors
═══════════════════════════════════════════════════

  Check Spine auth errors [BLOCKED]
  └─ 🔗 blocked by:
     └─ Upgrade HAProxy to 2.9 [IN-PROGRESS]

═══════════════════════════════════════════════════
```

## Part 9: The Weekly Review

The most powerful command. Run it on Friday or Monday:

```
> /review
```

Claude walks you through 8 sections interactively:

1. **Overdue items** — reschedule, complete, or cancel each one
2. **Stale in-progress** — tasks started 3+ days ago with no updates
3. **Waiting/Blocked** — any movement on these?
4. **Completed this week** — what you got done (morale boost)
5. **Time summary** — hours by project vs your daily target
6. **Upcoming deadlines** — next 7 days, anything need prep?
7. **Unscheduled high-priority** — should these get dates?
8. **Inbox check** — anything left to triage?

Takes 10-15 minutes. Leaves you with a clean, current system.

## Part 10: Export and Reporting

### Status report

```
> /export summary this week
```

Generates a markdown summary with completed tasks, in-progress work, time breakdown, and next week's priorities. Use it for standups, blog posts, or status emails.

### Timesheet export

```
> /export timesheet this month
```

Outputs CSV data grouped by project — ready for invoicing.

### Daily log

```
> /log today

═══════════════════════════════════════════════════
  📝 DAILY LOG — Thursday 26 February 2026
═══════════════════════════════════════════════════

  ✅ Completed
  ──────────
  1. [✓] Fix HAProxy connection timeout     ⏱ 3h15m

  ⏱ Time Logged: 4h
  ──────────
  (no project)    4h

═══════════════════════════════════════════════════
```

## Mobile Usage

Start a remote session on your desktop:

```bash
claude --remote
```

Scan the QR code on your phone. From there, you can:

- **Capture:** `/quick Remember to review the capacity report`
- **Check:** `/today`
- **Update:** `the Splunk task is done`
- **Triage:** `/triage`

The mobile experience is optimised for quick interactions.

## Configuration

Edit `~/clorg/.clorg/config.yaml` to customise:

```yaml
# Task states
states:
  default: [TODO, NEXT, IN-PROGRESS, WAITING, BLOCKED, DONE, CANCELLED]

# Known tags (for auto-matching)
tags:
  - nhs
  - infrastructure
  - haproxy
  - splunk
  - blog

# Time tracking
time_tracking:
  round_to_minutes: 5
  daily_target_hours: 6

# Auto-archive completed tasks after N days
archive:
  auto_archive_after_days: 7
```

## Tips

- **Context references work everywhere.** After any view, say "the first one", "#3", or "the HAProxy one" to act on items.
- **Natural language is first-class.** You rarely need slash commands — just describe what you want.
- **The weekly `/review` is the killer feature.** It takes 10 minutes and keeps the system alive. Do it every Friday.
- **Start simple.** Use `/add`, `/done`, and `/today` for the first week. Add time tracking and projects when you feel the need.
- **Your files are just markdown.** Open them in VS Code, Obsidian, or any editor. clorg-mode won't mind.
