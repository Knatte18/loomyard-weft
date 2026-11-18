---
verdict: BLOCKING
prior:
  - id: F14
    status: resolved
    answer: Card 5 now drops Command from its Edit list and has battenCLI.wire fill pickAccount, as wire fills catchUp beside its other seams. The CreateWorktree closure exists only inside wire, so every test that builds a battenCLI literal and reaches that closure goes through wire and finds a non-nil seam. The tests that set c.env.CreateWorktree directly never reach it. The card names TestWire_CreateWorktreePicksTheNewWorktreesAccount as the test that overrides the field after wire and asserts it non-nil.
  - id: F15
    status: resolved
    answer: Accepted with the fixer's different fix. finalize in internal/shuttleengine/wait.go has no idempotence guard and Wait opens with no terminal-record return, so the proposed Result{} and nil for a done relaunch that had already finalized would finalize twice. Card 14 now has startPath relaunches end through abandonStartup, which finalizes once and returns an error wrapping ErrNotStarted, or leave a done run to Wait. Card 14 also names a Start-driven row where the second Prepare fails. Card 16 states its loadRefusedResumeError path returns the same way. The stop-ended return of that same text is raised below as F17.
  - id: F16
    status: resolved
    answer: Cards 14 and 16 now list internal/shuttleengine/doc.go under Edit and name the sentences they rewrite. Each quoted sentence exists in doc.go, at the usage-limit paragraph (161), the re-send paragraph (190), the no-retries sentence (194) and the skill-load hold paragraph (570 to 574). No other doc.go sentence about usage limits, API-error re-sends or the load hold is falsified, and neither card lists a doc file under Extends.
findings:
  - id: F17
    severity: BLOCKING
    class: design
    location: _lym/plan/14-in-run-relaunch.md:65
    summary: Card 14 says a start-path relaunch that a stop ended returns the stop's outcome beside an error wrapping ErrNotStarted, but the stop's outcome is the record-only stopped value that is never a Result.Outcome, and no planned row reaches that branch through Start.
    fix: In card 14's load-turn paragraph, replace the sentence "A relaunch a stop ended returns the stop's outcome beside an error wrapping `ErrNotStarted`, writing nothing more." with a sentence stating that such a relaunch removes the strand, writes nothing to the record (it stays stopped), and returns a Result whose Outcome is OutcomeDied beside an error wrapping ErrNotStarted, the shape Wait's liveness read gives a strand a stop removed. Add that it leaves startupStrandRemoved false, so start's startRetryable never launches a stopped run again. Make the same change to the two other uses of "the stop's outcome" in card 14, in step 1 of relaunchUnderAccount and in the test row for a record a stop made terminal before the mark. Add a row to TestWait_UsageLimitSwitchRelaunchesInTheRun for a usage-limited skill-load turn driven through Start whose fakeReed OnEndStrandProcess hook runs StopStrand. Start returns no run and an error wrapping ErrNotStarted, the result outcome is died, the record reads stopped, no strand is up, and no second launch is made even with start retries set. No target or verify line changes, so no check in loom-plan-checks-spec.md constrains the fix.
---

### [BLOCKING] A stop-ended start-path relaunch returns an outcome that cannot be a Result and no row reaches it

**Location:** `_lym/plan/14-in-run-relaunch.md:65`

**Issue:**
Round 2's F15 fix added to card 14's load-turn paragraph that "a relaunch a stop ended returns the stop's outcome beside an error wrapping `ErrNotStarted`, writing nothing more."
The stop's outcome is `runOutcomeStopped`, which `internal/shuttleengine/stop.go:14-16` defines as a terminal value that "exists only in the run record, not in a reported result".
The package doc says the same: the record reads stopped, "a record-only terminal outcome that is never a Result.Outcome".
Where output files all exist, `stopOutcome` gives the record `done` instead.
So the card leaves open which `Result.Outcome` `launch` and `start` hand back, and an implementer who returns `stopped` breaks that contract.
The card also says "writing nothing more", so neither `finalize` nor `abandonStartup` runs to build the Result, and it names no outcome to build one with.
The same phrase is in step 1 of `relaunchUnderAccount` and in the row for a record a stop made terminal before the mark.
Those are Wait-path uses that earlier rounds left, and the fix covers them with the same wording.
Its only stop-race row, "a `StopStrand` run from `fakeReed`'s `OnEndStrandProcess` hook, between the mark and step 4", reads as a Wait-path row.
No planned row drives a stop through `Start`, so the start-path branch the F15 fix added has no named test that reaches it.
That branch must also not be retried: `start` retries a result that is `OutcomeDied` over a strand `abandonStartup` removed, and the card does not say the stop-ended relaunch leaves that flag false.

## Rejected

- `PATTERN.md` listed under card 14's `**Edit:**` while the card says its entry line is unchanged and the overview says the overview line stays as it is.
  It is a real inconsistency, but it is not a hunk this round's fixes changed and it is not BLOCKING, so an incremental round raises nothing on it.
- Card 5's Wiring paragraph says `wire`'s `CreateWorktree` closure calls `pickAccount` "with the slug and `keepEntry`" while the field's signature also takes the prime location.
  The closure's first argument is the location already in scope, so the call is unambiguous; not BLOCKING.
- Card 16 has no Start-driven row for a failing relaunch on its `loadRefusedResumeError` path.
  It states that the path returns as card 14's start path returns, and card 14's row reaches that return, so this is not raised separately.
  F17's fix covers the stop-ended branch for both.
- Card 14's sentence that a `startPath` relaunch "finalizes none of its died or timeout ends itself" reads against the next lines, where a non-done end goes through `abandonStartup`, which finalizes once.
  Read as the card's own lines 62 to 64 say, only the done end is left to `Wait`; a wording slip, not BLOCKING.
- Round 2's fixes to cards 5, 14 and 16 add no verb, flag or guard beyond what the decision record bounds: the pick no-ops are bounded by the card's own conditions, and the relaunch mark by the record's twice-startup-timeout bound in cards 12 and 15.
  Card order holds, since every symbol card 14 and card 16 use is created by an earlier card (cards 8, 11, 12, 13 and 14).
