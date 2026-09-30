---
round: 1
focus:
  - "D10 and D11, the clearing and resuming phases: D11 bounds only the handoff phase with a timeout and an abort path. Settle whether the decision record says what the watcher does when the startup-ready classification never arrives after the clear choreography, or when the resume turn never ends, and whether a restarted watcher in `clearing` can tell from observable state whether `/clear` already landed."
  - "D5 and D14 `stop` against D10's persisted phase: settle what happens to a non-idle `.lyx/orch/state.json` when `stop` removes the strand mid-cycle, or when the strand dies mid-cycle, and `start` then launches a fresh session. Does the fresh watcher resume the old phase into the new session (for example, typing a resume prompt or `/clear` into it), and does the record decide this?"
  - "D6 against D11: an aborted cycle can leave a handoff file that exists but is partial, or one written for a turn that never ended. Settle whether D6's 'newest file in `.lyx/orch/handoffs/`' can pick such a file as the resume source on the next fresh launch, and whether the record needs a rule that separates completed handoffs from aborted ones."
  - "D10 step 1 and D13 wording: D13 requires every injected stencil to render to a single line, and D10 describes the handoff injection as `/scribe:handoff <path>` 'with a focus line'. Settle whether these two statements are consistent as written, so the plan writer cannot read 'focus line' as a second line of input."
  - "D7 and D8, reading usage across `/clear`: D7 locates the transcript from what the Stop hook records, and the transcript changes at `/clear`. Settle whether the record makes clear what usage reading returns between `/clear` and the first post-resume turn end, and that a stale pre-clear reading cannot trigger a second cycle immediately after resume."
  - "D8 and D9 headroom: usage is checked at turn end, but injection waits for the idle grace and the input-box probe, and the handoff turn itself adds tokens. Settle whether the 150000 default leaves enough margin below Claude Code's auto-compaction point for the operator turns before idle qualifies plus the handoff turn, or whether Open risks should state that margin explicitly."
  - "Open risks versus Acceptance criteria: the smoke test is told to verify SendMessage address stability across `/clear` (risk 3), but not whether background tasks and their completion notifications survive `/clear` (risk 1, Q13). Background tasks are the premise that D9 rests on. Settle whether that risk also needs a verification step and a recorded outcome, or is acceptable as a mitigated open risk."
---

Every entry comes from reading the two artifacts, the writer's stencil, and a light check of the premises:
`EventWaiting` and `EventAsk` exist in `shuttleengine/engine.go` with the meanings D9 relies on,
`.lyx/` is gitignored as D12 assumes,
and `scribe:handoff` takes a path argument as D10 assumes.
None of these premises needs a focus entry.

Deliberately left out, per the rubric and the stencil:
the conditional scope item for a new shuttle read/type surface (interface shapes are Plan's to settle),
the absence of exact signatures or file:line citations,
the Notes for the plan writer section,
and the fact that the Auto-mode assumptions list overlaps with the D-entries, since that section exists to restate self-picks.
