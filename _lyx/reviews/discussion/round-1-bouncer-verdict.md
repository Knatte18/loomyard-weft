---
verdict: APPROVED
rationale: "All ten round-1 findings (F1-F10) are fixed in the current decision-record.md: D4 compares the (vcs.revision, vcs.modified) pair and the dev-build assumption matches, D1 confines text inspection to the Go failure site, D8 holds the in-flight resume rule and the spelled-out no-marker branch, D5 scopes the guard to the warp, D6 matches D10, Buildinfo Leaf is in Constraints, the SKILL.md criterion enumerates the skill-only behaviours, HANDOFF is in Scope Out, and Notes drop the Send parameter list; no new defect found."
---

# Discussion-Review round 1 — Bouncer verdict

The round-1 report returned `APPROVED` with ten findings (three MEDIUM, six LOW, one NIT).
The fixer report claims all ten fixed and none deferred.
I checked each claim against the current `decision-record.md` instead of taking the report's word for it:

- **F1**: D4 names the identity as the pair (`vcs.revision`, `vcs.modified`) and compares both fields.
  The last Auto-mode assumption now says successive dev builds from a still-modified tree at one revision compare equal, which agrees with D4.
- **F2**: D1's title bans text matching in the driver only.
  Its Decision allows the Go failure site to inspect the failed call's own output, with unrecognised output treated as not transient.
  The Open risks entry agrees with this.
- **F3**: D8 states the in-flight rule: no second step, and the line's report path and budget reset apply to the next stop.
  The Open risks entry now points at D8.
- **F4**: D5 scopes `git status --porcelain` to the warp, never the weft `_lyx` or `.lyx`.
  It also says a skipped re-step waits for the next binary change or a resume.
- **F5**: D6 starts the watch only when the stop is D5-eligible, which matches D10.
- **F6**: D8's live-driver-without-marker branch now says "send nothing, spawn nothing, then hand over", which matches its acceptance criterion.
- **F7**: Constraints lists the Buildinfo Leaf Invariant.
- **F8**: The `SKILL.md` acceptance criterion lists the D6, D7, D8, D9, D10 and D11 skill-only behaviours as sub-items.
- **F9**: `manifest/HANDOFF.md` is in Scope Out as operator-owned, with no Open risks entry left.
  Support-log Q15 records the move.
- **F10**: Notes write `shuttleengine.Runner.Send` with no parameter list.

The report has no `## Focus departures` section, so there are no departures to ratify.
Its "no finding" verdicts on focus entries 4, 7 and 8 hold.
I agree that the 2026-09-30 history in D5 is rationale, and that the Auto-mode assumptions record the autonomous picks as that section's contract.

My independent pass found no new finding:

- The support log's `## Review rounds` still reads `_No rounds yet._`.
  The writer stencil tells the writer to seed that exact line, and the gate appends round entries later, so this is not a defect.
- A spot-check of the named seams found `shuttleengine.ErrNotStarted`, `stepErrFields`, `driverTeardownCommand`, `startLLMDriverArm`, `internal/shedverbs/steprecord.go` and `internal/battencli/wire.go`, all where the record says they are.
- D3, D5, D6, D8 and D10 now describe one state machine with no contradiction:
  - one immediate transient re-step;
  - a binary-change re-step at the stop and on each deploy while parked, bounded by the recorded identity;
  - a resume that resets budgets and mints a fresh report path;
  - `done` and `busy` tearing down.
