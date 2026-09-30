---
round: 1
focus:
  - "Card 3 (add-remote-preflight): its Intent defines `probeWarpLeftover` and a shared ancestor helper that appear in no card's `Create:` list. Settle whether the card's targets fully name what it builds, or whether an independently testable symbol is missing from the plan's target surface (granularity rule)."
  - "Card 3's ImpactSummary says callers see a new typed refusal and no other change. `internal/hubforge/hub.go` calls `Topology.Add` with a zero `AddOptions{}`, so every hubforge-built hub in every package now runs the remote probes. Settle whether that conclusion covers the real set of `Topology.Add` callers."
  - "Fidelity, unarchived-leftover acceptance criterion: the decision record requires that no warp or weft worktree, branch, portal or launcher is left. Card 3's test enumerates the warp branch but not the weft branch. Settle whether every leftover the criterion names is asserted."
  - "Fidelity, destroy.go acceptance criterion: the record lists six unit-level cases for the extended request shape (unset ownership, unset dirtiness, wrong branch, checked-out pair weft not refused, unarchived tip, stale lease). Card 2 splits them between hermetic `destroy_test.go` and the integration test. Settle whether each case is carried by some test and whether any card adds cases the record does not license (e.g. the empty-lease-SHA refusal)."
  - "Fidelity, adopt path and warp side: check that Card 3's `probeWeftLeftover(localExists=true)` refuses a diverged remote even when an archive tag covers it (the record allows replacement only off the adopt path), and that no card deletes or force-pushes a remote warp branch."
  - "Card 5 tests the warp fast-forwardable case, which Card 3's probe implements. Settle whether that test placement is sound (it needs Card 5's weft replacement to reach success) or whether it hides a Card 3 behavior from Card 3's own tests."
  - "Card 5 edits `rollbackAdd`'s doc comment and adds a package-level test hook var. Settle whether its ImpactSummary is a real blast-radius conclusion covering the rollback semantics change (the remote deletion is never restored), not a restatement of Intent."
---

The plan is five cards: a leased `gitrepo` delete, the `destroy.go` gate extension, the Add pre-flight probes, the CLI envelope, and the step-12 replacement plus docs.
The `## verify:` section runs `go test ./...` and the integration tag over all three targeted packages (`gitrepo`, `fabricengine`, `fabriccli`), so verify coverage looks complete on first read. It is not in focus.
Handle declaration/reference balance, path existence and glyph spelling are the gate's job and are left out.
