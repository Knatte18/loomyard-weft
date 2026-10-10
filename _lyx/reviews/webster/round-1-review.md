---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: cmd/lyx/testdata/test-budget.yaml:86
    summary: the budget rows for shedengine, proc and loomcli were raised past the top-level test count the diff leaves in each package
---

### [LOW] Budget rows raised past the tests the diff adds

**Location:** `cmd/lyx/testdata/test-budget.yaml:86` (`internal/shedengine`), `:73` (`internal/proc`), `:55` (`internal/loomcli`).

**Issue:** The file's own header says to raise a row "in the commit that adds the tests that need it".
Counted the way `cmd/lyx/testbudget_test.go` counts (top-level `func Test…` in every `_test.go`, `TestMain` excluded), the packages stand at:

| package | tests now | row now | row before |
|---|---|---|---|
| `internal/shedengine` | 133 | 135 | 130 |
| `internal/proc` | 17 | 18 | 12 |
| `internal/loomcli` | 182 | 183 | 182 |

The diff adds three top-level tests to `shedengine` (`TestStep_PauseBeforeCondition`, `TestStep_PauseAfterCondition`, `TestWriteFailedStop`), so its row carries two spare slots.
`proc` carries one spare slot.
In `loomcli` the diff adds `TestWire_MissingHubLandingConfigNamesWayForward` and deletes `TestRemoveStatusStrands`, so the count is unchanged and the raised row is a spare slot.
The other raised rows (`shedverbs` 79, `pairteardown` 6) match their counts exactly.
A spare slot lets the next test in that package land without a budget edit, which is what the budget exists to force.
The plan's cards state these row values, so the finding is about the diff's bookkeeping, not the plan.

**Fix:** Set `internal/shedengine` to 133, `internal/proc` to 17 and `internal/loomcli` to 182.

## Focus verdicts

- **Windows bodies (cards 1, 2, 12, 18).**
  I ran `GOOS=windows CGO_ENABLED=0 go vet ./internal/proc ./internal/gatecli ./internal/pairteardown ./internal/shedverbs`: clean.
  The diff does not record that each card ran its own `**Verify:**`, but the result it names holds on the final tree, so no finding.
  The Windows bodies match the Linux semantics for the job tie (kill-on-close job per child, a named kill-on-close job per loop) and the tree kill (`TerminateJobObject` on the child's job, `TerminateNamedJob` on the loop's).
  The one difference is `Quit`, a documented no-op on Windows, so the watchdog there waits out `quitGrace` before it kills.
  That follows from the design and is not a defect.
- **Loop body, detach and wait, watchdog (cards 11, 12, 13).**
  Stop classification (`halted`, `stop-condition`, `error`, `interrupted`, `busy`), the single transient re-step, the failed write with its `Busy` turn, the dead-loop stop and the idle kill behave as the plan states.
  `TestLoop_*` and `TestUntilStop_*` drive every behavior untagged through `stepCmd` over the `LoopSpec` seams (`Runner`, `Spawn`, `Watch`, `Sleep`, `Now`, `Activity`, `TraceFiles`), and `TestLoopIntegration_ChildTreeDiesWithLoop` is the one real-child test.
  The plan's `TestLoop_UnarmedSpecRefused` is covered by the `unarmed spec` row of `TestLoop_RefusalsBeforeAnyStep`.
- **Engine stop conditions and `WriteFailedStop` (cards 4, 5).**
  `pause_before` and `pause_after` clear together only in the persist that records `paused` for a fired condition and in `Goto`; a bare pause and every other write carry them through.
  `WriteFailedStop` takes the run lock without waiting, reads the file strictly under it and writes only while it reads `running`.
- **Loop files told through `Spec.Loop` (cards 3, 15).**
  `internal/shedverbs` imports neither `internal/shedrun` nor `internal/lyxcwd`, and `seam_enforcement_test.go` lists exactly the allowlist the background file states.
  It joins paths only under the told `StepsDir`, as `steprecord.go` already did.
- **`PATTERN.md` cap (cards 8, 9, 11, 12).**
  `wc -c PATTERN.md` is 16137, under 16200 and equal to the plan's figure after card 12.
  Every dropped overview clause is held by its background file: the three new background files (`batcher-registry`, `parent-directive`, `shuttle-stop`) state it, and the others (`no-denied-recovery`, `gogit-read-helper`, `review-round`, `agent-name`, `comment-line-breaks`) already did.
- **Rename and Delete groups (cards 9, 16, 20).**
  The old names `TestStepKinds_IsExactlyFive`, `TestArmFromSeed_MissingRunRefusalCarriesNoKindField`, `TestWarnPortBackDrift_EmitsClassAndRemedyPerDifferingStencil`, `removeStatusStrands` and `TestRemoveStatusStrands` have no reference left outside `docs/benchmarks/test-redundancy.md`, a measurement report.
  The new names exist, and `reconcileOne` has the signature card 20 states.
- **Test economy.**
  I found no duplicated test, no per-helper test already covered by a public-surface test, and no unpinned added behavior.
  Package tests (`shedverbs`, `shedengine`, `proc`, `pairteardown`, `gatecli`, `shedrun`, `logger`, `loomshed`, `stencilstore`, `contracts/stencils`, `cmd/lyx`, `shedcli`, `loomengine`, `internal/pattern`) pass, and so does `--tags integration` over `shedverbs`, `pairteardown`, `proc` and `loomcli`.
