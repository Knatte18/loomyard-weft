```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, listing per change the set found and whether the diff covers it.

**Per change: set found, covered or not**
- `PATTERN-slug-remote-order` (add) and `PATTERN-push-both-sides` (change) in `PATTERN.md`: both match the declared text. Covered.
- `pattern/PATTERN-fabric-git.md` and `pattern/PATTERN-push-both-sides.md`: both gain a bullet describing the new archive order. Covered; the first edit is the subject of F3.
- `originTargets` gained a return value: callers are `originreconcile.go:725` and `destroy.go:1898`. Covered. The `DropLocal` call leaves `PairMayBeAbsent` unset, so D1's boundary holds.
- `removeWeftWorktree` and `removeSibling` gained `leaseSHA`: callers are `remove.go:537`, `weftwiring.go:234` and `add.go:852` (rollback, passes ""). Covered.
- `archiveWeftTip` and `archiveTaskTip` gained a `sleep` parameter: callers are `remove.go:281`, `cleanup.go:320` and `taskbranchdiscard.go`. Covered.
- `deleteWeftBranchOnRemote` gained `leaseSHA`: callers are `cleanup.go:332`, `remove.go:519` and `weftwiring.go:234`. Covered.
- `lockPairTeardown` gained `rerun`: callers are `add.go:245`, `remove.go:219`, `taskbranchdiscard.go:38`, `cleanup.go:299` and `cleanupremotewarp.go:149`. Covered.
- `refuseAdoptLocalCheckedOut` and `probeWarpLeftover` changed signatures: the only callers are `add.go:386` and `add.go:442`. Covered.
- `reconcileVerdict`, `reportVerdict` and `stoppedReason` gained `followUp`: callers are `parentalign.go:139`, `finalize_owner.go:38` (nil, the D3 exception) and `finalize_owner.go:51`. Covered.
- `removeFields` gained `warning`: `fabric.go` only. Covered.
- `PushPairAnchored` now returns `ErrPairPushRunning`:
  - `fabriccli/weft_verbs.go:299` and `loomcli/commitrecords.go` turn it into an ok envelope with `push_skipped`.
  - `loomcli/wiring.go:219` is the `Push` field behind `commit-records`, so it is covered there.
  - `loomcli/landingdeps.go:338` returns it unhandled; see F2.
- Docs in `fabricengine`, `fabriccli`, `gitrepo`, `loomcli`, `landingshed`, `pairteardown` and `tendercli` were updated. My grep for the old "only `--adopt-local`" and "not present at" wording found no stale copy outside `battencli/wire.go`, which the diff rewrote.
- D3 sites naming `lym fabric remove`:
  - Covered: battencli cancel and owner preflight, landingshed, tendercli, `lym fabric remove`'s own warning.
  - Left unchanged, see F1: `battencli/wire.go:319` and `fabricengine/failedpush.go:233-236`.
- Commit messages: none of the range's commits carries a `Pattern-Departure:` trailer, so there is no departure to judge.

**Findings**

- **F1 — MEDIUM, scope.** `internal/battencli/wire.go:318-319`, `internal/fabricengine/failedpush.go:233-236`.
  - D3 says every stop or refusal naming `lym fabric remove <slug>` for a slug with a batten record names the step that settles the record.
  - `incompletePairRemedy` tells the operator to run `lym fabric remove --force <slug>` on an interrupted create.
    - The batten record then stands at Worktree-Create and is left standing.
    - It gets no `lym batten cancel <slug> --remove` follow-up.
  - The failed-push "gone" way forward (`failedpush.go:233`) tells the operator to decide whether the run ends with `lym fabric remove`, and also names no settling step.
  - The card-13 follow-up wiring (`pairteardown/followup.go`, `landingshed`, `battencli` owner and cancel, `tendercli`) does not reach either site.

- **F2 — LOW, consistency.** `internal/loomcli/landingdeps.go:338`.
  - This synchronous `PushPairAnchored` call returns the new `ErrPairPushRunning` unhandled when a removal, Add or discard holds the teardown lock.
  - Its sibling at `loomcli/wiring.go:219` is covered by the `commit-records` skip.
  - I did not read the code that consumes `landingdeps.go:338`, so whether the error becomes a halt is unconfirmed.

- **F3 — LOW, consistency.** `pattern/PATTERN-fabric-git.md:32`.
  - The diff edits this entry's background file, but the record's PATTERN changes list only `PATTERN-slug-remote-order` and `PATTERN-push-both-sides`.
  - The edit describes behaviour the code now has.
  - `PATTERN.md` itself does not change for `PATTERN-fabric-git`, so the one-line entry is untouched.
  - The record has no explicit licence for the edit.

Tool calls (6):
1. `grep "lym fabric remove"` over `internal` and `contracts`: lists every site naming the removal as a way forward, which my context lacked.
2. `grep` of the changed symbols' callers: my context held only the diff.
3. `grep` for old wording ("only `--adopt-local`", "not present at"): looks for stale docs.
4. Read `battencli/wire.go:296-326`: context for `incompletePairRemedy`.
5. Read `loomcli/wiring.go:205-235`: how `PushPairAnchored`'s new error is consumed.
6. Read `fabricengine/failedpush.go:222-241`: the "gone" way-forward text.
