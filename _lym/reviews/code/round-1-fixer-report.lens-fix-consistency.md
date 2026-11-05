```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for stale twins, through the fix-consistency lens.

**Findings**

1. **LOW** — `internal/pairteardown/doc.go:53-54`, beside the new `RemoveResult.StrayPathDeleted` set in `internal/pairteardown/teardown.go` (`clearStray`).
   - **Summary:** the module doc says what `ClearStray` deletes and when a delete fails. It never says `RemovePair` now reports the delete in `RemoveResult.StrayPathDeleted`.
   - **Evidence:** the doc was extended by the earlier teardown commits for `ClearStray` and `ErrStrayPathKept`, but F6 adds a result field, the one outcome a caller reads, with no sentence about it. A reader of the doc cannot tell that `cancel --remove` depends on that field for its `stray_directory_deleted` output.

2. **LOW** — `internal/fabriccli/fabric.go:1134` and `fabric.go:1163` (explicit field map), `internal/fabriccli/removefields_test.go`.
   - **Summary:** `RemoveResult.StrayPathDeleted` has a JSON tag but the fabric remove envelope never maps it.
   - **Evidence:** the envelope lists "appear only when set" fields by hand, and `stray_path` is mapped at line 1163. The new field is not. This is harmless today because `lym fabric remove` never sets `ClearStray`, but it leaves the result type and its CLI renderer out of step.

Checked and consistent:
- The `PATTERN-fabric-git` amendment and the `doc.go` destruction-chokepoint paragraph carry the same two-exception text.
- The `removeWorktreeLeftover` doc is back to its narrow wording.
- The `cancelcmd.go` help `Output:` list and `cancelEnvelope` both carry `stray_directory_deleted`, in the same position.
- `remove.go`'s respawn guard is back to its original form, which agrees with the F5 and F6 commit messages.

Tool calls:
1. grep for stray-directory wording across `battencli`, `pairteardown/doc.go`, `PATTERN-pair-teardown.md`, `fabriccli` and `contracts`, to find docs or twins not shown in my context.
