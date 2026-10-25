# Fixer report — discussion round 1

F9 and F10 are `decision`-class findings and went to the parent; this report leaves them out.

## Changes

- **F1** (fixed as proposed). D7: each accepted entry records who accepted it, the row's live agent session with its attempt, or a caller at a `Drift-Verify` halt.
  D6: the scope-`full` Stuck on a passing compare fires only on the current attempt's agent acceptances, and names `lyx loom resume`, after any `lyx drift accept` of the parent's own, as the way forward.
  The fix also adds `lyx drift withdraw` so the parent can reject an acceptance.
  D2 and D6: under scope `full`, every drift halt is a decision halt, its reason opening with `shedadapters.DecisionHaltPrefix`, which `arbiterstop.HandsBack` marks, so the stop chain hands it to the parent ahead of the tackler and the arbiter.
  That makes the arbiter unable to accept under `full`, and the `lyx drift accept` refusal the fix offered as an alternative is not needed.
  Checked against `internal/arbiterstop/arbiterstop.go:61-63`, `internal/sheddriver/doc.go:30` and `internal/shedengine/run.go:394-400`.
  The acceptance criterion is rewritten as proposed, with the decision halt added.
- **F2** (fixed as proposed). D4 now ends a timed-out command through `proc.StartTied` and `Tied.Kill` on both platforms, citing `internal/proc/tied.go` and `internal/gatecli/child.go` `runTiedChild`.
  Removed: the Windows hold sentence, Scope In's `runproc`-entry bullet, the `## PATTERN changes` section, and the PATTERN exception in the first Constraint.
  Support-log Q12 is marked superseded.
- **F3** (fixed as proposed, judgement branch). D7 now says that under `partial` an acceptance can let through every test the compare runs, a compare left with no test included.
  It names the agent's judgement as the only bound and says the arbiter may accept under `partial`.
  The enforced bounds the fix suggested are not possible: lyx cannot read `baseline.md`'s freeform expectation or the harness's test list.
- **F4** (option b). D4's main-capture guard now also admits a halted `Drift-Await`.
  The way forward in D5 is `lyx drift capture` then `lyx loom resume`.
  D3 says why `lyx loom goto Drift-Baseline` is not used in loom.
  The line-188 acceptance criterion is updated to match.
- **F5** (fixed as proposed). A D5 paragraph covers the adopt case: the old machine's capture is ended, and the adopting machine has no capture record or baseline.
  Before `awaited_at` the row halts and the run recaptures on the branch point; after `awaited_at`, `Drift-Verify` halts naming `lyx loom goto Describe`.
  The "or an adopt on another machine" clause is dropped, and an acceptance criterion is added.
- **F6** (fixed as proposed). Drift-Await no longer has a time bound while the lock is held.
  D4's monotonic-clock sentence is kept.
  The timed-out case in the acceptance criteria now reads the `failed` record, and the line-184 criterion is aligned with it.
- **F7** (fixed as proposed). D6: `lyx drift compare` and `lyx drift capture --side` refuse unless the current row is `Drift-Verify`.
  The counter resets at each fresh spawn of the row's agent, and the row's own mechanical compares never count.
  Acceptance criterion updated.
- **F8** (option a, carried out through a new verb). D5's ways forward now say the regression question is deferred to a second halt at `Drift-Verify`.
  In the HEAD-moved case the passed baseline is compared.
  The self-check (way-forward 2) showed that `lyx loom goto Webster` skips `Drift-Await` and leaves `awaited_at` unset, so every Webster reroute would halt again.
  The way forward is therefore `lyx drift defer` then `lyx loom resume`: `defer` writes `awaited_at` as a deferral and refuses unless the run is halted at `Drift-Await`.
- **F11** (fixed as proposed). D4 now says the plan gate's type load, also a slot holder, may queue behind the capture.
- **F12** (fixed as proposed). D6 limits the claim to fixes the agent or a gate re-prompt makes.
  D2 adds Finalize's merge and merge fixer, and darn's Finalize reroute, as uncompared.
- **F13** (fixed as proposed). Scope Out now assigns the sandbox harness to note `csharp-sandbox-hub` and makes the Models run on OSL-1046 the final confirmation.
  The plan-writer note is reworded to match.
- **F14** (fixed as proposed). The first acceptance criterion is rewritten as given.
- **F15** (fixed as proposed). D4 line 93 now states the exclude as an existing fact.

## Self-check

- fix-consistency, MEDIUM, a capture run by a caller is unregistered and runs in the foreground (merged with fix-proposal 1): fixed with own change. D4's caller-run capture starts the same detached, registered child as the row and returns once the child holds the lock. An acceptance criterion is added.
- fix-consistency, LOW, D7's accepter sentence offers the arbiter under `full`: fixed as proposed. It now reads "the arbiter under scope `partial`".
- fix-consistency, LOW, the ledger misses the new auto-picks: fixed as proposed. Support-log Q13 to Q15 are added.
- fix-claims, LOW, `DecisionHaltPrefix`'s doc names only the bouncer halt: fixed as proposed. A plan-writer note says the constant's doc, or a shared hand-back marker, widens to cover the Drift halts.
- fix-way-forward, MEDIUM, a withdrawal does not hold against re-acceptance: fixed as proposed. Withdrawn tests move to a withdrawn list, and the agent session's `lyx drift accept` refuses them. Acceptance criterion updated, ledger Q15.
- fix-way-forward, MEDIUM, `goto Webster` leaves `awaited_at` unset, so reroutes halt again: fixed with own change, `lyx drift defer` (see F8).
- fix-way-forward, LOW, the adopt criterion covers loom only (merged with fix-proposal 3): fixed as proposed. D5 and the criterion now cover darn.
- fix-proposal, MEDIUM, a caller-run capture dies with the caller through Pdeathsig: fixed with own change (see the first self-check line).
- fix-proposal, MEDIUM, an escaped descendant holding the output pipe blocks `Wait` and the lock forever: fixed with own change. D4 makes the open log file itself the command's stdout and stderr, so no copying pipe keeps the capture from recording `failed` and releasing the lock. The old wall-clock bound in `Drift-Await` stays dropped.
- fix-proposal, LOW, the darn adopt case is unstated: merged into fix-way-forward LOW above.
- fix-proposal, LOW, the no-baseline halt under `full` is no decision halt, so the arbiter could goto `Describe`: fixed as proposed. D6 makes the no-passed-capture halt under `full` a decision halt too.

## Disputed

None.

## Deferred

None.
