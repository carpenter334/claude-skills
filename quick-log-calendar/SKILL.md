---
name: quick-log-calendar
description: >
  Quick-log time spent on work activities directly to your Google Calendar.
  Use this whenever you want to log what you just finished working on — 
  say something like "log 30 min on SAP documentation" or "I just spent 2 minutes 
  on a quick call" and this skill creates a calendar event on your James 
  Carpenter calendar with cherry blossom color, marked as free time, and set 
  to private. Accepts any duration (2 min, 45 minutes, 1.5 hours, etc.). 
  By default it logs the last 30 minutes ending now, but you can specify 
  different durations. Trigger on: "log", "quick log", "add to calendar", 
  "time entry", or casual phrasing like "I just worked on X for the last Y".
compatibility: Requires Google Calendar connector (Google Workspace MCP).
---

## Quick-Log Calendar

This skill creates a calendar event from a single sentence about what you just did, matching your time-tracker format (cherry blossom color, private, free).

### How it works

When you give a task like "I just spent 45 minutes on the SAP integration", the skill:
1. Extracts the activity description and duration (defaults to 30 min if not specified)
2. Accepts flexible duration parsing: "2 min", "30 minutes", "1.5 hours", "half an hour", etc.
3. Calculates the time window (end = now in your local timezone, start = now - duration)
4. Creates a Google Calendar event on **James Carpenter (primary)** with:
   - **Title**: "[Time tracker] " + your activity description (e.g., "[Time tracker] SAP documentation")
   - **Color**: Cherry blossom (colorId: 10)
   - **Visibility**: Default
   - **Show as**: Free
   - **Start**: now minus the duration
   - **End**: now

### Input format

Just tell Claude what you did and how long (defaults to 30 minutes if not specified):

**Examples:**
- "Log 2 min on email"
- "I spent 45 minutes on customer calls"
- "Worked on Boomi integration for the last hour and a half"
- "Quick log: Prepared Q4 presentation — 90 min"
- "45 min on SAP docs"

### Implementation approach

Use Google Calendar API to create events with these parameters:
- `calendarId`: "primary"
- `summary`: "[Time tracker] " + activity title extracted from user input (e.g., "[Time tracker] SAP documentation")
- `startTime`: Get current time NOW, then subtract the duration to calculate start time (ISO 8601 format with timezone)
- `endTime`: Current time NOW (ISO 8601 format with timezone) - this is when the work ended
- `timeZone`: User's local timezone (auto-detect or use America/Chicago)
- `colorId`: "10" (cherry blossom - verified from test results)
- `visibility`: "default"
- `availability`: "AVAILABILITY_FREE"
- `description`: leave empty

**IMPORTANT:** Always use the actual current time (NOW) for the end time, and calculate start time by subtracting the duration from NOW. Never use hardcoded or estimated times - the event must span from (NOW - duration) to NOW.

The "[Time tracker]" prefix clearly identifies these as time tracking entries in the event title.

Confirm after creation: "✓ Logged 30 min on SAP documentation (11:22–11:52 AM)"

Use Google Calendar connector to:
1. Parse the user's natural language input to extract activity title and duration
2. Calculate start and end times in the user's timezone
3. Create the event with the specified properties, including "time tracker" in the description
4. Return confirmation with the logged time range
