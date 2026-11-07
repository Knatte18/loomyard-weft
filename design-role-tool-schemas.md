# Tool schemas are three quarters of a spawned role's start; drop the tools a role never calls

`role-tool-schemas` · Task · research, shuttle, claude · running · Drift-Baseline

## Seen

`docs/research/tokens/session-scope-fix-launch-2026-11-06/report.md`: a spawned role's first request is 29,087 tokens with the session scope on, and its tool schemas alone are 21,947 of them, about three quarters.
The role's tools are `Agent`, `Artifact`, `Bash`, `Edit`, `Read`, `ReportFindings`, `ScheduleWakeup`, `SendFeedback`, `Skill`, `ToolSearch`, `Write` and `advisor`.
The orch's tool schemas are 9,927 tokens; why a role's are more than twice that is not known.
The operator finds about 30k tokens at start too much (2026-11-07).

## Wanted

A report, and no change to code or config: the operator decides which tools to drop from it before any is dropped (operator, 2026-11-07).
It lands under `docs/research/tokens/`.

- Each tool's schema size in tokens, in a spawned role's first request and in the orch's, from recorded transcripts with the existing `measure.sh`, and why a role's tool schemas are more than twice the orch's.
- For each role lym spawns, which tools its sessions called, counted over the transcripts on this machine, so a tool no role calls stands out.
- Which way of dropping a tool removes its schema from the request: `--disallowedTools` on the launch line, or `permissions.deny` in the settings lym passes.
  Settle it with at most two launches in the sandbox hub `lyt` through `lym shuttle open`, one spawned role each, `opus[high]`, dropping `ScheduleWakeup` one way in each.
- A table per role: tool, schema tokens, calls seen, and whether it can go, with the tokens a role's start would lose.
