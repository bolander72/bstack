---
name: caldir
description: "Manage calendars and events via caldir CLI — view today's schedule, list upcoming events, create new events, sync with iCloud and Google Calendar. Use when user asks about their schedule, what's on the calendar, upcoming events, creating meetings/appointments, checking availability, or anything involving calendar management. Also use for 'what's today look like', 'am I free on Thursday', 'schedule a meeting', 'add an event'."
---

# caldir — Calendar Management

Project: https://caldir.org/

CLI tool for managing calendar events across iCloud and Google Calendar. Events stored locally as .ics files, synced bidirectionally with remote providers.

## Setup

- Binary: `~/.local/bin/caldir` — always export PATH first:
  ```bash
  export PATH="$HOME/.local/bin:$PATH"
  ```
- Config: `~/Library/Application Support/caldir/config.toml`
- Events stored in: `~/caldir/{calendar-name}/`

## Core Commands

### View events
```bash
caldir today          # Today's events
caldir week           # This week through Sunday
caldir events         # All upcoming events
```

### Create an event
```bash
caldir new            # Interactive event creation
```

Or write a .ics file directly to `~/caldir/{calendar-name}/` and push.

### Sync
```bash
caldir pull           # Remote → local
caldir push           # Local → remote
caldir sync           # Both directions
```

### Other
```bash
caldir status         # Check for changes (local and remote)
caldir config         # Show config paths and calendar info
caldir discard        # Discard unpushed local changes
caldir update         # Update caldir and providers
```

## Calendars

Calendars are directories under `~/caldir/`. Each contains .ics event files.

Common calendar names (these are directory names, may vary by setup):
- Personal/default iCloud calendar
- Shared calendars (family, partner)
- Google Calendar (work)
- Holiday calendars

Check available calendars: `ls ~/caldir/`

The default calendar is set in config.toml via `default_calendar`.

## Creating Events via .ics

For programmatic event creation, write a .ics file directly:

```ics
BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//caldir//EN
BEGIN:VEVENT
DTSTART:20260415T140000
DTEND:20260415T150000
SUMMARY:Team standup
DESCRIPTION:Weekly sync
LOCATION:Zoom
UID:unique-id-here@caldir
END:VEVENT
END:VCALENDAR
```

Save to `~/caldir/{calendar-name}/{filename}.ics`, then `caldir push`.

### Date/time formats in .ics
- All day: `DTSTART;VALUE=DATE:20260415`
- Local time: `DTSTART:20260415T140000`
- With timezone: `DTSTART;TZID=America/Chicago:20260415T140000`
- UTC: `DTSTART:20260415T190000Z`

## Workflow: Check Schedule

1. Sync first for fresh data: `caldir pull`
2. View events: `caldir today` or `caldir week` or `caldir events`

## Workflow: Create Event

1. Either `caldir new` (interactive) or write .ics file to the target calendar directory
2. Push to remote: `caldir push`
3. Verify: `caldir today` or `caldir events`

## Workflow: Check Availability

1. Pull latest: `caldir pull`
2. List events for the target date range: `caldir events`
3. Look for gaps in the schedule

## Tips

- Always `caldir pull` before reading events to ensure freshness
- Always `caldir push` after creating/modifying events locally
- Use `caldir sync` when you want both directions
- The `caldir new` interactive mode handles timezone and recurrence nicely
- For bulk event creation, writing .ics files directly is faster than interactive mode
- UID in .ics files must be globally unique — use a UUID or `{descriptive-slug}@caldir`
