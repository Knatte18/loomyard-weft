```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review of the plan's four cards.

Findings:

1. LOW, Card 3 (D1 re-add) and Card 2 (bootstrap add): the race guard only matches a strand under its full current name. A strand still under the legacy bare name `loom-status` slips past it.
   - Evidence: `internal/reedengine/strand.go:600-605` skips only when `strandNamed(st.Strands, unlessFull)` finds the exact full name that `validateNaming` builds.
   - Both cards' own lookups (`findStatusStrand`, the `agentname.Matches` loop in `internal/orchcli#orchCLI.ensureRunsStrand`) also accept a legacy exact name.
   - In practice the gap can't be reached on the paths the cards change:
     - In `ensureRunsStrand`, the legacy strand found is the one just replaced.
     - In `ensureStatusStrandOn`, a found legacy strand takes the keep or replace branch and never reaches the add.
   - So no second strand results. The cards' claim that "a strand holding the loom status role" makes the add skip is broader than what reed checks. The fix is to word the Intent as "a strand holding the full `loom-status` name". Severity is LOW because the cards rely on nothing more.

Every other guard and row the cards add is reachable from a real entry point:
- Card 3's held skip: `lym loom start` reads strands after beam start's replace and then adds. That is the bootstrap-lock-free window between `ReplaceStrand` and `AddStrandUnless`.
- Card 2's held skip: the same race seen from the loom side.
- `replaceErr`: a concurrent loom start replaces the same stale guid first.
- The done, absent and unreadable self-run cases.
- The sidecar-blocking rows: a directory at the path makes `os.WriteFile` fail after `MkdirAll` succeeds.
- Card 4's no-`reed.json` row: `LoadState` returns `(nil, nil)`, which reaches `checklistWindowName(nil)`.
- Card 4's garbage `reed.json`, garbage status and garbage seed rows.

The guarded site doesn't recur unguarded elsewhere. Every `BeamUp` caller passes the target worktree's own location: `beamcli/cli.go:248`, `loomcli/pawlwiring.go:107`, `adoptcli/deps.go:133`, and `BeamUpWorktree` through `adoptcli/green.go:246`.

Tool calls:
- Read `internal/reedengine/strand.go:600-630` to see how `AddStrandUnless` matches the unless-name; my context held only its doc comment. In the same call I grepped the `BeamUp` callers to confirm each passes the beam's own location, since my context held only the decision record's claim.
