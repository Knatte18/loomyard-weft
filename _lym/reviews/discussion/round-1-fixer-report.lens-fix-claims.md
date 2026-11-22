```yaml
lens: fix-claims
outcome: reported
```

Checked the fix's code claims against HEAD: two of them are wrong.

**MEDIUM: D9 names the wrong config flag for the orch's read**
- Changed text: D9, "its own worktree's value from `lym config beam --print`, run when it takes a D7 step".
- Summary: `--print` prints one layer's file as written, not the resolved value, so an orch reading it misses a `true` set on the default layer. `lym config beam --resolved` prints the resolved value.
- Evidence: `internal/configcli/doc.go:30-31` says "`--print --layer <name>` prints that layer's file verbatim" and "`lym config <module> --resolved` prints the merged result through `configengine.Provenance`".

**MEDIUM: D3's way forward claims resume behaviour the code does not have**
- Changed text: D3 Bound, "the halt names `lym loom resume`, which asks the parent again".
- Summary: the judge's ask code sends a new ask only when the judge's rulings change. Nothing shows a resume re-asking the parent about the same rulings after the deadline passes. The record states this as existing behaviour when it is new and unnamed.
- Evidence: `internal/driftengine/judgeparent.go:102` says "an open ask is set timeout, and a new ask naming the new rulings goes out in its place". Lines 216-221 judge the ask against its deadline, and no path re-asks about unchanged rulings.

These claims hold:
- D9's list of pointers matches every `WithPresence` caller:
  - start and reload: `internal/orchcli/start.go:836,854`
  - the pending resume after a handoff: `internal/orchengine/watcher.go:1281,1600`
  - the reload after compaction: `internal/orchengine/watcher.go:1351`
- `internal/shellguard/rows.go` has no row naming `selfreport` or `parent-review`.
- A new pawl run starts without the earlier run's files: `internal/pawlengine/beamup_test.go:29` asserts that Beam-Up clears what an earlier run left.

Further calls:
1. `internal/configcli/doc.go:1-35` and `internal/orchengine/watcher.go` around the `WithPresence` calls, plus a grep of `internal/shellguard/rows.go`, to check the `--print`, pointer and shellguard claims, which my context had not read.
2. A grep of `internal/driftengine` for the judge timeout, and of `internal/pawlengine` beamup for clearing, to check the D3 and D6 claims.
3. `internal/configcli/doc.go:22-31` and the head of `internal/driftengine/judgeparent.go`, to confirm what `--print` prints and how the judge's ask works.
4. A grep of `internal/driftengine/judgeparent.go` for when an ask is sent again, to check the resume claim.
