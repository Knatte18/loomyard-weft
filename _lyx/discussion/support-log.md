## Interview

Autonomous session: no operator answered; every pick is an auto-pick in the question ledger.

Turn 1 — read the board entry `twb-ftb-3` and the board list.
Related entries: `arbiter-fixes` (owns the implementer `deviations` paragraph), `weaver` (reuses `RenderRecoveryPrompt` and `uncommitted_paths`), `glyph-use-audit`, `logging-cleanup` (no overlap).
`twb-ftb-1`, `twb-ftb-2`, `twb-fta-1`, `twb-fta-2` are sibling bench runs of the same entry, not dependencies.

Turn 2 — read issues #505, #509, #517, #522, #511, #500, #507 and the base `800a0e892` code.
Finding: at base no background shell expires; the remaining premature-done path is `countsAtOnce` admitting a fresh gated run's first arrival when every outstanding task is an unawaited shell, and Master declares only `lyx webster recover-batch` as awaited.
Finding: shuttle's generic gate re-prompt tells Master to "fix every finding", contradicting its stencil.
Finding: `recordedSessions` reads Master sessions only, so a recovery strand's writes are never attributed.
Finding: `recover-batch` returns exit 0 for any terminal digest that is not batch_failed.
Finding: the recovery prefix names no allowed gates.
Finding: quarry resolves unexported and same-package test-file identifiers found; `#511`'s `rusageStats` landed beside `rusage_linux.go`/`rusage_other.go`, so its failure most likely was per-GOOS ambiguity.
Finding: `DoneChecks` accepts only found or multipart, and the Plan-time status policy blocks any ambiguous answer.

Turn 3 — settled D1 to D7 (see the decision record) and the out-of-scope list.

Turn 4 — lens review: all six forks reported; duplicates merged across lenses.

- scenarios BLOCKING + premises MEDIUM + pattern-fit MEDIUM — D6 let partitioned glyphs pass Plan while `planGatePass` and `answerSymbols` still skipped ambiguous answers: fixed, a partitioned answer now reads as multipart with its candidates as symbols at every site, plus a delete-order acceptance row.
- premises LOW — `statusFindings` is not handed the worktree root, and `DetectDrift` also calls it: fixed in D6 and the told-geometry constraint.
- board-fit LOW — D6's GOOS model ignored implied tags (`android`→`linux`, etc.): fixed, D6 now uses `go/build`'s tag implications, with an `android` acceptance row.
- scenarios LOW — `_test.go` mixed with non-test candidates: fixed, fail closed.
- leanness MEDIUM + pattern-fit LOW + premises LOW — D5 duplicated the body's `batch_gate`: fixed, the section points at the body; A7 dropped.
- scenarios MEDIUM + goal-scope LOW — D5's skipped tmux steps had no stated checker: fixed, bound names `go vet -tags tmux`/`llm` in the plan verify and Publish's `publish_verify`, which a base read of the plan stencil confirmed; A8 records the unexplained premise.
- scenarios MEDIUM — D3 left the prior record's own id, attach and unknown session id unstated: fixed, with acceptance rows.
- scenarios MEDIUM — D1 had no transcript-only row and no never-seen-completion risk: fixed (acceptance row, R2).
- scenarios LOW — D1 delays a paused outcome: fixed, accepted as R3.
- scenarios LOW — D2's malformed re-prompt effect unstated: fixed in D2's bound and the acceptance row.
- scenarios LOW — other `recover-batch` exit readers (`llm-model-spec.md`): fixed, Scope names any spec describing its exit.
- goal-scope MEDIUM — D4 did not show which path exited 0 on a "failed" batch: fixed, A5 records the reading, and an acceptance row pins `batch_failed`'s non-zero exit.
- goal-scope LOW — the Goal overstated item 6's Plan-time flag: fixed, the Goal says the flag is informational.
- goal-scope LOW — the Notes missed `drift.go` and `plangate.go`: fixed.
- pattern-fit LOW — D2 presented the stencil's no-edit rule as new: fixed, D2 says the stencil already states it.
- pattern-fit LOW — the Go line named a stencil heading: fixed, it names the prompt file only.
- pattern-fit LOW — `webstercli`'s package doc missing from docs: fixed.
- premises LOW — R1 claimed the settings refuse a leading `cd`: fixed.
- board-fit MEDIUM — `arbiter-fixes` may edit `webster-template-master.md`: fixed, named in Constraints.
- board-fit LOW — `weaver` inherits D3/D4: rejected, `weaver` is parked and nothing contradicts; no line needed.
- board-fit LOW — sibling bench entries: no change needed, already treated as siblings.
- leanness MEDIUM — rejected alternative in D2's rationale and A1–A3's arguments: fixed, both cut to one line.
- leanness LOW — D7 narrative, quarry stated three times, sole-parsers constraint, duplicate refusal-row criterion, Notes duplicates, long Goal: fixed.
- leanness LOW — the "still counts at once, as today" criterion: fixed, dropped as existing coverage.

## Rejected alternatives

- D1, stopping the gate shell and recording its result as discarded: lyx has no pid for a provider shell, and stopping throws away the result Master needs.
- D1, matching `lyx gate test` anywhere in the label instead of as a prefix: changes shuttle's prefix contract for a case the agents' settings mostly refuse.
- D2, a verb that spawns the verify fixer: a fork can only be spawned by the Agent tool inside Master's session; a verb would spawn a cold strand at recovery cost.
- D2, stencil wording only: the generic re-prompt would still contradict the stencil at the moment Master reads it.
- D3, starting the second recovery from a clean tree with the first attempt's diff as reference: needs a reset of uncommitted work and discards progress the prompt already knows how to keep.
- D3, attributing every uncommitted path at respawn to the batch: no evidence, would turn operator or other-batch files into the batch's own.
- D6, changing quarry to honour build constraints: quarry is an external module in another repository.
- D6, accepting any ambiguous answer at the done check: would pass a real duplicate declaration.
- D6, a blocking Plan-time finding for a partitioned glyph: would keep `#507`'s Edit impossible.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): hold the gate shell or stop it? Hold, via an awaited prefix (D1, A1).
- Q2 (auto-pick): message fix or new verb for the verify fixer? Message fix with a caller-supplied re-prompt (D2, A2).
- Q3 (auto-pick): attribution by batch or clean tree for the second recovery? Attribution by batch (D3, A3).
- Q4 (auto-pick): does a dead digest with a retry left exit non-zero? Yes, the envelope carries `recovery_retry` (D4, A4).
- Q5 (auto-pick): what does "the plan validator flags such a glyph at Plan time" mean for a Create? An informational finding for an existing partitioned glyph; a Create cannot be predicted at Plan time (D6, A6).
- Q6 (resolved by reading): do unexported and test-file identifiers resolve? Yes, found; `#511` traced to D6's cause (D7, A7).
- Q7 (resolved in the lens review): are the recovery's gates rendered again in the prefix? No, the prefix points at the body's sections, which already render them (D5).
- Q9 (auto-pick): which path exited 0 on a "failed" batch in `#522`? Read as a terminal stuck or dead digest (A5).
- Q10 (open): why could the recovery strand not run tmux-tagged steps when forks run them through `lyx gate test`? Not established; D5 follows the board entry (A8).
- Q8 (auto-pick): do reset and the run-exit audit also read recovery sessions? No, outside the entry's scope.
