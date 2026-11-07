# A Codex seat that read its skills is logged as not having loaded them

`codex-skill-load-false-missing` · Task · bug, shuttle, codex · done

## Seen

In the Code-Review of `assist-fan-seat-work-alone` on 2026-11-06, the Codex assistant seat's trace logged `shuttle: skill skipped ... cause="not loaded"` for all five skills at 13:29:48, after both load turns, with the network up.
Its rollout, `~/.codex/sessions/2026/11/06/rollout-2026-11-06T13-29-27-01a1b0d5-*.jsonl`, shows an `exec` `custom_tool_call` naming `SKILL.md` at 12:29:33Z, so the seat seems to have read its skills and the warning looks false.

`custom_tool_call` is among `toolCallPayloads` in `internal/shuttleengine/codexengine/skillload.go`, so the payload kind alone is not the cause.
The cause is not found: the classifier may miss the skill tail inside the `exec` input, or the turn it reads may not be the one holding the call.

## Why it matters

A false warning hides a true one, and the task `skill-load-timeout-sends-prompt` leans on the same load reports.

## Wanted

Find why the classifier read these skills as not loaded, and make it read a Codex load correctly, or show that the skills were in fact not loaded.
