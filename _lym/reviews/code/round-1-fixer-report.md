---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The premise holds, but neither proposed option covers every listed state. Option (a), probing origin's records branch, still refuses once nothing of the pair remains (F5's state), and option (b) keeps a `lym fabric remove` that refuses there too. Instead `lym batten cancel <slug> --remove` counts an unfinished teardown whose record names it as something left, which covers every state the record is listed in."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "The proposed test needs a teardown child that ends, and the seam for that is in fabricengine's export_test.go, which another package cannot reach. The closure's body moved into `removeThrough`, and a tier-0 test pins with a fake remover that the request's Rerun reaches the origin removal."
  - id: F9
    disposition: fixed
---

## What changed

- **F1, F5:** `refuseNothingLeft` (`internal/battencli/cancel.go`) counts, under `--remove`, an unfinished teardown whose record names this cancel.
  When the cancel's removal answers `ErrPairNotFound`, it calls the new `Topology.SettleGonePairTeardown`,
  which pushes the slug's archive tags origin lacks and drops the record.
  The cancel's help text and `ErrNothingLeft`'s comment say so.
  The discard now builds its re-run string with `cancelCommand`, so the record match cannot drift from it.
  Tests: two rows and one extended row in `TestCancelRun`, `TestCancelIntegration_FinishesAnUnfinishedRemoval`,
  and the settle half of `TestRemove_PendingRecordNamesTheRerunItsCallerGave`.
- **F2:** removed the battencli sentence from `internal/fabricengine/doc.go` and added one on `SettleGonePairTeardown`.
- **F3:** taken through the fix key's "unless" branch: the cancel now finishes in that state, so the condition stays dropped.
  The comment in `recordsArchiveWayForward` is reworded, and `Remove`'s doc states what a caller's rerun command must do.
- **F6:** "four-argument" is now "five-argument" in `pattern/PATTERN-pair-remote.md`.
- **F7:** new subtest `RemoveWhoseWaitRunsOutNamesTheGivenRerun`.
- **F8:** `removeThrough` in `internal/pairteardown/teardown.go`, tested in `removethrough_test.go`.
- **F9:** `_lym/landing/summary.md` has the wider blast radius, the corrected bullet and the new behaviour and tests. It is not committed here.

`lym gate test` on `internal/battencli`, `internal/fabricengine` and `internal/pairteardown` passed at tier 1 with no failures.
The tier-0 run also passed; only comments changed in Go files after the tier-1 run that preceded the self-check.

**Gate repair:** the round's comment gate rejected a wrapped comment on `ErrNothingLeft` and two over-long lines
on `SettleGonePairTeardown`.
Both comments are rewritten as one sentence per line under the length limit; no code changed.

**Second gate repair:** `TestCLIForm_EveryCommand` in `cmd/lym` failed, since the sentence added to the help of `lym batten cancel`
took its `Long` to 2030 bytes against a cap of 2000.
The sentence is now a short parenthesis in the refusal list.
`lym gate test ./cmd/lym ./internal/battencli` passed at tier 0.

**Third gate repair:** `TestLandNotice_ReachesAnOrchStrandListedAfterTheLanding` in `internal/dobbyengine` failed.
The branch had not touched that package before this repair.
The failure reproduced 3 times in 500 runs.
The test's last clock advance wakes the delivery workers, and one still probing once the orch strand was listed
typed the notice into its pane before the test read it.
The test now stops the scheduler, which waits for those workers, before it lists the orch;
no assertion changed.
It passed 3000 runs of 3000, and the package passed at tier 0.

## Self-check

- fix-consistency, LOW, `cancel_integration_test.go` file header omits the new case: fixed as proposed.
- fix-consistency, LOW, summary's `## Tests` bullet omits the settle half of the test: fixed as proposed.
- fix-affected-set, LOW, `PendingTeardown` type doc contradicts the settled state: fixed as proposed.
- fix-code-correctness: no findings.
- fix-proposal: no findings. Its aside, that a cancel's later discard can overwrite a failed removal's record, predates this fix and was not checked.

## Disputed

None.

## Deferred

None.

## Rulings

None.
