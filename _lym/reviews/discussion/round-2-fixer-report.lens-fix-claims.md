```yaml
lens: fix-claims
outcome: reported
```

Task: check, under the fix-claims lens, that each sentence the fix added or rewrote holds against the code at HEAD.

1. **MEDIUM.** D5, the new bullet "`Coverage.Held` is set for it, so the dobby notice, the batten overview and the tender report show it as a hold."
   - Problem: this claim about what reads `Coverage.Held` comes from the reviewer's proposal, and the code does not hold it.
   - Evidence: the only production reader of `Coverage.Held` is `internal/battencli/dobbyreads.go:179`. It feeds `RunView.Held` (`internal/dobbyengine/passes.go:56`), which the dobby's owner probe reads (`passes.go:193`).
   - `tenderengine`'s report has a `Held` field, but it is `[]HeldBranch` of branch holds (`internal/tenderengine/report.go:203`) and is not filled from pause coverage.
   - No batten overview reads `Coverage.Held`.
   - So setting `Held` for a `bolt-hold` entry reaches only the dobby's run view. It does not reach the three display readers the sentence names.

2. **No finding** on these claims; each holds:
   - **D1:** the anchor change's push, fabric's commit spawn (`internal/fabriccli/spawn.go:14`) and the lift and teardown respawn paths (`teardownlock.go:169`) all go through `SpawnStatusPush`. Its child runs `CoalescePushRequest`, which reaches `coalescePushSides` (`teardownlock.go:111`, `coalesce.go:98-99`). That step pushes warp before weft (`coalesce.go:152-164`), so "today pushes the code side first" holds.
   - **D3:** `machinename.Names` returns the current and former names (`machinename.go:83`). The no-directory variables match `noDirectoryRefusal` (`internal/machinecli/set.go:88-94`).
   - **D5:** `pauserecord.CoveredError` is used by loom (`loomcli/pausehold.go`, `driverresume.go`) and shuttle (`shuttleengine/open.go`, `pausedsend.go`), so "a loom or shuttle refusal" is grounded.
   - **D6:** `lym adopt check` runs from the fix task's landing through the `adopt-check` row: `fronttask.ActionAdoptCheck` is an `OnLand` key (`internal/fronttask/fronttask.go:21-33,60-61`).

Tool calls:
- Grep for the callers of `CoalescePushRequest` and `CoalescePushBothAt`, and the uses of `SpawnStatusPush`: my context lacked which detached paths reach `coalescePushSides`.
- Grep for `.Held` readers, `CoveredError` users and the `adopt-check` `OnLand` row: my context lacked who reads coverage and refusals.
- Grep for `Held` fields in the dobby, batten and tender packages, plus the spawn child's command: my context lacked the shape of `tenderengine`'s `Held`.
- Grep for where `coverage.Held` is consumed: my context lacked the one reader, `battencli/dobbyreads.go`.
