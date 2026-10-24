# no-5hr-limits

Starts the Claude 5-hour usage window automatically from the cloud, so the PC can stay off.

## What this is

Claude subscriptions meter usage in 5-hour sessions. The clock starts with your first message and resets 5 hours later. The limit is shared across claude.ai, Claude Desktop and Claude Code, so a message from any of them starts the same window.

If a window is already running when you sit down to work, it resets sooner, and you get a fresh one partway through the day instead of one window tied to whenever you first opened Claude. This project sends a tiny "hi" to Haiku every 5 hours on a Claude cloud routine, so a window is always counting down.

There is no code here. The cloud routine is the whole system, and this file documents it.

## Current setup

| Field | Value |
|---|---|
| Routine name | `no-5hr-limits ping` |
| Routine ID | `trig_01CQUABzxHJNnGQYG3oBd3Cj` |
| Link | https://claude.ai/code/routines/trig_01CQUABzxHJNnGQYG3oBd3Cj |
| Type | Claude Code cloud routine, no repository attached |
| Environment | `Default` (`env_01Mzy6XPEKiKg6GVVSxrUx2Q`, anthropic_cloud) |
| Model | `claude-haiku-5-5` |
| Prompt | `hi. Reply with only the word "hi". Do not use any tools.` |
| Cron (UTC) | `7 2,7,12,17,21 * * *` |
| Fires at (Istanbul, UTC+3) | 00:07, 05:07, 10:07, 15:07, 20:07 |
| Created | 2026-10-24 |

Routine cron expressions are always in UTC. Turkey has no DST, so the conversion is always +3 hours and never needs seasonal edits.

## How the schedule plays out (Istanbul time)

| Ping | Window already active? | Result | Covered until |
|---|---|---|---|
| 00:07 | Yes, the 20:07 window runs until 01:07 | No new window | 01:07 |
| 01:07 to 05:07 | No | Uncovered, about 4h overnight | |
| 05:07 | No | New window 05:07 to 10:07 | 10:07 |
| 10:07 | Just expired | New window 10:07 to 15:07 | 15:07 |
| 15:07 | Just expired | New window 15:07 to 20:07 | 20:07 |
| 20:07 | Just expired | New window 20:07 to 01:07 | 01:07 |

24 hours isn't divisible by 5, so one gap each day is only 4 hours (20:07 to 00:07), and the 00:07 ping lands inside an active window. Windows roll continuously from 05:07 to 01:07.

If you start a window yourself (say at 09:30), the next ping that lands inside it does nothing, and the first ping after it ends opens a new one. The schedule corrects itself within one cycle.

## Managing it

- **Pause / resume:** the on/off switch at https://claude.ai/code/routines (or the Desktop app, Code tab, Scheduled).
- **Run it now:** Run now on the routine page, or `/schedule run` in Claude Code.
- **See past runs:** the routine's History tab, or ask `/schedule` about its run history.
- **Change times:** edit the routine and set a custom cron, or use `/schedule update`. Remember to enter UTC: Istanbul hour minus 3.
- **Change the model:** edit the routine, or use `/schedule update`.
- **Delete:** only possible on the routine page at https://claude.ai/code/routines. Deleting it stops all pings.

## Caveats

- About 4 hours overnight (01:07 to 05:07) are uncovered.
- Each ping uses a small amount of quota. It runs as a Claude Code cloud session, which carries a system prompt, so it costs more than a bare chat message but is still small on Haiku.
- While you are at your usage limit, runs are rejected until the window resets.
- A paused or lapsed subscription puts routines on hold. You have to turn them back on yourself.
- Routines are a research preview. The feature, its limits and any daily run caps may change. Old third-party reports mention caps of 5 runs a day on Pro. This routine uses exactly 5.
- The rule that the window starts at your first message comes from an older support article. Current articles only say "resets every five hours". See the verification below.
- The server attached default tools and two default connectors to the routine. The prompt tells it not to use any tools, so they go unused.

## Verifying it works

1. **Run now, then History:** the newest session should show Haiku replying `hi`.
2. **Window check:** when you haven't used Claude for 5 hours, run it once, then open claude.ai, Settings, Usage. The current session's reset time should be about 5 hours after the run. If you've used Claude recently, the run lands inside your existing window and proves nothing.
3. **Schedule check:** after a day, History should show runs near 00:07, 05:07, 10:07, 15:07 and 20:07 Istanbul time, a few minutes late at most.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| No runs in History | Routine is off, or the subscription is paused | Turn the switch on, check the subscription |
| Runs fire 3h off | Cron was entered as local time, not UTC | Use `7 2,7,12,17,21 * * *` |
| A run shows as rejected | You were at the usage limit | Expected. The next ping after the reset works. |
| Usage page shows no new window after a run | The run landed inside an existing window, or the concept doesn't hold | Compare the run time with the window end |
| Haiku no longer available | Model list changed | Switch to the cheapest model offered |

## Fallback if routines go away

A GitHub Actions workflow on a cron schedule can do the same job. It runs `npx @anthropic-ai/claude-code -p "hi" --model haiku` and authenticates with a token from `claude setup-token`, stored as the repo secret `CLAUDE_CODE_OAUTH_TOKEN`. GitHub cron often starts 10 to 30+ minutes late.
