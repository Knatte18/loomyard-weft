# Fixer report — plan, round 1

## Changes

- **F1 (BLOCKING), fixed differently.**
  The review proposed putting the cuts in card 10, the last card to edit `PATTERN.md`.
  I did not do that: the verify gate measures overview growth from the merge base at every gate (`internal/loomshed/verifygate.go:103-108`), so card 8's 73-byte rewrite would fail its own batch before card 10's cuts landed.
  Card 8, the first card to grow the overview, now has an "Overview headroom" paragraph that shortens eight entry lines with their meaning kept: told-geometry, mutation-record, shed-recipe-registry, plan-generation, creel-store, completion-signal, spawn-observability and documentation-lifecycle.
  The cuts total about 168 bytes against 157 of growth, so `PATTERN.md` ends card 8 about 95 bytes under its base and card 10 about 11 under.
  Both declared texts stay verbatim.
  Each background file already states the detail its line drops, so no `pattern/` file is edited.
  Card 10 states that its rewrite fits inside that room.
  Card 8's ImpactSummary and the overview's card index line for card 8 mention the cuts.
- **F2 (MEDIUM), fixed as proposed.**
  `sessionFakeEngine` gains `markEvents`, which is set only by the new `TestRunner_ReadEvents` rows.
  With it set, `Event` goes only on `STOP:`, `WAIT:` and `ASK:` lines, and a bare `ASK` stays unmarked.
  `internal/shuttleengine#sessionFakeEngine` is added to Edit.
- **F3 (MEDIUM), fixed differently.**
  The review proposed an inert default `ParseSessionSignals` on `shuttlefake.Engine`.
  I did not do that, because shuttleengine branches on the capability itself, not only on the signals it yields: `send.go:608` (the landing wait), `sessionread.go:79` (`SessionCauseUnsupported`) and `waitshadow.go:105`.
  An inert parser would still move every consumer of the kit onto those branches.
  Instead `Engine` stays without the parser, and a new `SignalEngine` in `shuttlefake.go` embeds `*Engine` and carries the stamp reading.
  `TestEngine_InertDefaults` now asserts that `Engine` implements no `SessionSignalParser`.
  The "no test of its own" sentence is gone, and the ImpactSummary is rewritten.
- **F4 (MEDIUM), fixed as proposed.**
  In card 10, every tick that finds a pending step with no id runs `adoptPending` before it sends, after a failed call as well as after a restart.
  A new `TestWatcher_CreelFailuresHoldTheCycle` row covers an inject that is journaled but answered with an error.
  `plan:internal/orchengine#fakeCreel.Inject` is added to Edit.
- **F5 (LOW), fixed as proposed.**
  `injectCmd` is added to card 4's Edit, and it passes `--timeout` only with `--wait`.
  The ImpactSummary now says "every non-waiting CLI call".
- **F6 (LOW), fixed as proposed.**
  `./internal/pattern` is appended to both verify `lyx gate test` lines.
- **F7 (LOW), fixed as proposed.**
  Those refusals carry no code, so `start` appends one fixed way-forward text to them.
  That text also names the two causes: run from the orch's pane or from outside every strand pane, and restart or remove a strand whose pane process is unknown.
  A new `TestStart_Refusals` row covers the program-claim refusal.
- **F8 (LOW), fixed as proposed.**
  The hold refusal names its way forward (`lyx creel post`, `lyx orch distill` or `lyx orch refresh`), and the refusal rows assert the full text.
- **F9 (LOW), fixed as proposed and then refined by the self-check.**
  `creelFailed` now also sets a persisted `State.StuckByCreel` (`stuck_by_creel`).
  A tick in which every creel call succeeded clears that stuck reason.
  A new test row covers a failed release at idle followed by a clean tick.
  Card 11 states that `FreshLaunchFailure`'s stuck reason is not cleared by that rule.
- **F10 (LOW), fixed as proposed.**
  Card 4's outside-pane `TestDaemon_Protocol` row runs on Linux only.
- **F11 (LOW), fixed as proposed.**
  Card 2's Intent gains D1's Bound line.

`lyx loom validate-plan` passes.

## Self-check

- fix-proposal, MEDIUM: clearing `Stuck` on the next successful call flaps when one call fails every tick beside one that succeeds. Fixed differently: the clear now happens only after a tick in which every creel call succeeded, not after any one successful call.
- fix-proposal, LOW: "a Stuck that creelFailed set" needs a source marker. Fixed: `State.StuckByCreel`.
- fix-consistency, MEDIUM: the "outnumber" `TestRunner_ReadEvents` row could no longer be built once marks pair one to one. Fixed: a `MARK` line, read with `markEvents` set, gives a marked signal that yields no event, and the row uses it.
- fix-consistency, LOW: the shuttlefake package doc goes stale with `SignalEngine`. Fixed: card 1 updates the package doc.
- fix-claims, MEDIUM: `SignalEngine`'s per-line reading is meaningless over the default `ParseEvents`. Fixed: card 1 requires a test to set `ParseEventsFn` to a per-line reading, and the type's doc comment states it.
- fix-affected-set: no findings. Its note that card 8 rewords entry lines the decision record does not declare is not a finding: the cuts change wording, not invariants, and validate-plan's pattern-changes-mismatch check passes.

## Disputed

None.

## Deferred

None.
