# Webster round 1 — fixer report

Every finding was validated against the code and the plan, and all four are fixed, each in its own commit.
`lyx gate test` passed over `internal/websterengine`, `internal/loomshed`, `internal/shuttleengine` and `cmd/lyx` in the unit tier and in the `integration` tier.
The gate's comment lint then refused one line the F3 commit added, and a follow-up commit repairs it, described under F3.

## Fixed

### F1 — reworded post-rebaseline refusals have no test (BLOCKING)

Commit `2303557f0`.
The finding holds for both texts it names, but only the quarry transient can be reached through `Run`.
`TestRun_AutoRebaseline`'s plan-validation subtest became a table, and a new row reaches the quarry transient after an accepted rebaseline by pointing `WorktreeRoot` at a missing tree.
The row asserts `ErrAutoRebaseline`, `planglyph.ErrQuarryUnavailable`, the warning lead, `way forward: transient, <re-step> once quarry answers`, neither manual verb, and no Master start.

No plan reaches the step-keyed zero-batch refusal in `recordedBatches` after an accepted rebaseline, so no `Run` row can reach it.
A plan with no card fails to load, with `planparser`'s `no card index entries found`; I tried the row and got that error.
`ExecutionBatches` then maps a recorded partition that covers every card through `mapPartition`, or forms one identity batch per card.
The card prescribes the step-keyed `zeroBatchesError(planDir, step)` signature, so I kept the branch and pinned both of its texts in the new `TestZeroBatchesErrorText` in `findingtext_test.go` as golden strings.
A comment on the `Run` table says why that table has no zero-batch row.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/websterengine` row rises from 198 to 199 for the one new top-level test.

### F2 — `TestEditedDoneCards` repeats a gate row (LOW)

Commit `7df69927f`.
I applied both halves of the suggested fix.
`TestEditedDoneCards` drops its begun-batch fixture and assertion and keeps the cases nothing else covers: a done batch's recorded id the plan lacks, and a nil state.
`TestNewPlanGate`'s row `EditedUnbegunCardReportsNothing` is renamed `EditedInFlightCardReportsNothing` to match what it builds.

### F3 — empty waiting list changes the ungated at-once rule (LOW)

Commit `3d014e03e`.
`pollEventsTick` now counts an ungated `EventWaiting` with an empty outstanding list as a Done when every output file exists, before it calls `expiredTurnEnd`.
`expiredTurnEnd`'s empty-list guard stays, because the no-new-bytes ticks use it to tell that no waiting turn end is recorded.
A gated empty list still holds, as card 22 requires, and `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` already pins that.
The ungated case gets a new row in `TestPollEventsTick_OutstandingBackgroundWork`, and `pollEventsTick`'s doc comment states the case.
The new call counts as a second `allOutputFilesExist` site in `pollEventsTick`, which tripped `TestCompletionSignal_FileContractCallSites`.
The new site is a positive Done that consults the files, so I raised `auditedFileContractCallSites["pollEventsTick"]` to 2 with a justification line, as that test's message directs.

That justification line first ended its previous sentence on a comma, and the gate's comment lint refused it as a fixed-column break at `completionsignal_enforcement_test.go:185`.
A follow-up commit ends that sentence with a period and makes the new site's line a sentence of its own.

### F4 — stale "waited out" comment (NIT)

Commit `d74af4f37`.
I reworded `loopAgreesWithState`'s reason to what still produces the busy-on-background pairing: a turn end counted at once, or a plain Stop after a waiting one.
Both leave the session running tasks the loop no longer waits on, so the `loopHeld` and `loopDone` allowances are still reachable and stay.

## Disputed

None.

## Deferred

None.
