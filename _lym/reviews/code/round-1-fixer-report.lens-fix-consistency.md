```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fixer's change set, looking for twins left unchanged.

**LOW**: `readNoticeChild` now returns `PairDir` with `found == false` (F9), but the seam contract it implements was not updated.
- Evidence: `internal/dobbyengine/runnotice.go:109` says only "found is false while the child has no status file".
- The run watch now relies on `child.PairDir` being set on a not-found read.
- Only the comment on `readNoticeChild` in `internal/battencli/dobbynotice.go` says so.

**LOW**: F20 added the nested-run exception to `Holder.Marker`, but its twin still states the unqualified claim.
- Evidence: `internal/gatecli/run.go:806` says `runMarkerEntry` "returns the environment entry that every process the tier run of pid starts carries".
- A nested `lym gate test` replaces that entry, because it uses the same variable name.

**LOW**: the torn-read change (F8) leaves the `RunWatchState.StatusSeen`/`ChildSeen` comment slightly stale.
- Evidence: `internal/dobbyengine/runwatch.go` says they are the modification times "at the last child read".
- A successful read of a child written after the listing now records neither. They are the times at the last accepted read.

Nothing else disagreed:
- The key-name sweep found no bare key left in added non-test Go lines.
- Both help texts with the done-run removal sentence were updated.
- `contracts/orchrole/orch-role.md` still agrees with the gate-hold behaviour.

Tool calls (3 of 6):
1. A grep of the branch diff for bare key names, plus docs and contracts for done-run and process-tree wording. My context lacked the whole-branch result and the docs text.
2. A read of the `ChildReading` and `ReadChild` docs in `runnotice.go`, plus a line-length check of added comments. My context never showed those declarations.
3. A grep of `gateslot/doc.go`, `gatecli/run.go` and the `orch-role.md` lines for marker wording. My context did not show them.
