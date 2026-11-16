# Round 3 fixer report — plan

All five findings are fixed.
`lym loom validate-plan` returns `ok: true` after the fixes and after the self-check changes, with informational findings only.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Card 5: `AttemptState` and `FixerState` gain `Reentry`, set on the first attempt a re-entry starts and the first fixer a verify re-entry spawns.
  Card 12: `runBatch` takes a `reentry bool` parameter and marks its first attempt `Reentry` when it is set.
  The lost-attempt allowance counts the lost attempts on the rung from the batch's latest `Reentry` attempt on, or from its first attempt when none is marked.
  The lost-fixer allowance counts fixer records the same way.
  Card 13: the `resumeStrands` step 4 allowance uses the same count, and the begin bullet passes `reentry` to `runBatch`.
  The verify re-entry's fixer is marked `Reentry`.
  The `TestRun_Reentry` row for a lost re-entry attempt now covers a batch stopped after two last-rung losses.
  A verify row covers the matching fixer case.
- **F2 (MEDIUM), fixed differently in part.**
  Card 12: on entry, after the plan-conflict record is removed and before `resumeStrands`, `Run` removes `outcome.yaml` and `summary.md`.
  Card 13's `Run` sequence names the same removal.
  Card 23's Intent says `RunInFlight` relies on it.
  The proposed `TestRun` row asserted `weaverengine.RunInFlight`, which card 23 creates after card 12, so card 12's test cannot call it.
  The new row asserts instead that neither file exists while the new batch's attempt runs, and that Done writes both again.
  `TestRunInFlight` in card 23 already covers an absent `outcome.yaml` reading as in flight.
- **F3 (MEDIUM), fixed as proposed.**
  Card 9 creates `batchGateCalls`, which returns the `card` calls in card order and then the `batch` call.
  `batchGateSpec` wraps those calls in its closures and appends the row's `gates`.
  Card 13 uses `batchGateCalls` and `runplan.IsLastBatch`.
  `checkStoppedBatch` runs exactly those calls through `RunDeps.Verify`, with `last` taken from `IsLastBatch` over the recorded partition mapped onto the current plan.
- **F4 (LOW), fixed as proposed.**
  Card 12 creates `outcomeRecord` and `writeOutcome`, and `Run` writes `outcome.yaml` through `writeOutcome`.
  Card 23 uses `outcomeRecord`: `RunInFlight` decodes into it and refuses unknown keys.
  The sentence that restated the keys is dropped.
- **F5 (LOW), fixed as proposed.**
  Card 12 says each batch stop, needs-parent stop and verify stop sets `st.Stop` and saves it through `saveRunState` before the Stuck return, a failed-stop Stuck included.
  A plan-conflict Stuck and a config or state fault set none.
  `stopReason` is stated to write no state.
  The `TestRun` row for a last-rung failure asserts the saved `Stop`.

## Self-check

- fix-consistency and fix-proposal, MEDIUM (merged): a crash during a re-entry calls `reenter` again and marks a second `Reentry`, which resets the count.
  Fixed: card 5's `StopRecord` gains `Reentered`, which the save that records the marked attempt or fixer sets.
  Cards 12 and 13 mark `Reentry` only while `Reentered` is unset.
  A `TestRun_Reentry` row covers a crash during the retry.
- fix-consistency and fix-proposal, MEDIUM (merged): `resumeStrands` step 4 stops the row, but card 12 named only `runBatch` and `Run` as writers of `Stop`.
  Fixed: card 12 names `resumeStrands` for a stop it judges from a recorded attempt.
  A `TestRun_Resume` row asserts the saved `Stop` after a crash following a last-rung failure.
- fix-claims, LOW: `gateslot` has no `--wait` main-packages call.
  Fixed: card 9's `batch` call is `GateCall` with `" --" + gateslot.MainPackagesFlag` appended when `last` is set, and card 9 uses `MainPackagesFlag`.
- fix-affected-set, LOW: card 12 left the other Stuck routes unstated.
  Fixed: a verify runner crash and a fixer whose stop failed are verify stops set by `Run`.
  A failed per-batch commit or final record commit sets none.
- fix-affected-set, LOW: card 8's prompt renders the gate calls separately from `batchGateCalls`.
  Disputed, premise false; see below.
- fix-affected-set, LOW: `seedWeaverState` restates `outcome.yaml`'s keys.
  Disputed, every fix worse; see below.

## Disputed

- Self-check (fix-affected-set) item 1, **premise false**.
  Card 8's prompt and the strand's gates are not two hand renderings that can drift.
  Both come from the same `gateslot` key and card scope, in the two forms `gateslot` defines on purpose.
  `internal/gateslot/tiers.go:376` says `GateCall`'s `--wait` form "is for a call Go runs".
  `AgentGateCall` at `tiers.go:382-386` is "the call an agent runs", and it carries no `--wait`.
  The prompt has to show the agent form and the Go verify site has to run the `--wait` form, so neither can render the other's calls.
- Self-check (fix-affected-set) item 3, **every fix worse**.
  `seedWeaverState` sits in `internal/fabriccli_test`, so it can only reach `outcomeRecord` if `weaverengine` exports it, and that export would exist only for a test (PATTERN-test-only-code).
  The fixture writes the file's raw bytes, as `seedWebsterState` does (`internal/fabriccli/mergescenario_integration_test.go:130-142`, called at `:542`).
  That pins the on-disk contract at the boundary the merge-in scenario tests.

## Deferred

None.

## Rulings

None reached this round.
