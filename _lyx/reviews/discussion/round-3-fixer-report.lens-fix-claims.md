```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every claim the fix added or rewrote against the code at HEAD.

**MEDIUM: D10's rewritten rationale says every path outside D7 that can change a checked-out marker refuses or stops, but `lyx adopt --drop-local` does neither.**
- Changed text: "every path that can change a checked-out marker outside D7, checkout, a weft pull, a replaying weft push, the origin reconcile's weft moves and promote, refuses or stops before changing anything … and D8 and an adopt write only D5's value".
- Evidence: `internal/fabricengine/destroy.go:1855`, `DropLocal`, "moves the code and records branches of the pair slug to the tips origin last held for them". Each move is a `reset --keep` in the branch's own worktree, its records (weft) worktree included.
- It is reached from `internal/adoptcli/adopt.go:345-346`, the `req.DropLocal` case of `takeOverSlug`. It runs whenever the pair is not live, and it tests neither the marker nor a move record.
- If another machine re-anchored the pair and pushed, origin's weft tip carries a different marker. `--drop-local` then resets the checked-out weft to that tip, so the marker changes while the junctions stay at the old anchor: the half-rewired pair D10's rationale says no path leaves.
- The clause "an adopt write[s] only D5's value" is therefore also false for the drop-local route. The move-record refusal on adopt in D7 covers only a standing record, not a marker change made on another machine.

The other claims hold:
- `ReconcileOrigin` runs hub-wide and run-scoped (`originreconcilemoves.go` `move`, which checks `m.scope.Run`).
- Its weft moves go through `reconcileRecordsBranch`, which calls `ffWorktree` or `replayWeft`. The move of the code branch is `ffWorktree` on the warp worktree and touches no marker.
- Per-branch stopped outcomes already exist (`BranchBusy`, `BranchLive`, `BranchDiverged` carrying `WayForward`), so "reports the branch stopped … never fails its caller" fits the existing shape. `reconcileHub` in `internal/orchcli/start.go` returns the report.
- `internal/lock` records no holder, so the record's new pid field has no rival source.

Tool calls:
- grep for `reset`, `switch`, `checkout` and `ResetHard` in `fabricengine` and `gitrepo`: to look for other movers of a checked-out weft, which my context lacked.
- grep for `DropLocal` callers: to find who reaches the `reset --keep` mover.
- `sed` of `destroy.go:1835-1905` and `adopt.go:330-360`: to read `DropLocal`'s behaviour and the guards on its call site.
- `sed` of `internal/orchcli/start.go:235-300`: to check how `orch start` consumes the reconcile report for the "never fails its caller" claim.
