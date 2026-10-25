# Card 19 — `lyx adopt` refuses while an anchor change stands

**Edit:**
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#adopt`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli/doc.go`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli#world.deps`

**Uses:**
- `internal/fabricengine#MoveInProgress`
- `internal/lyxcwd#NamedWorktree`

**Intent:**
D7's adopt refusal.
`adoptDeps` gains `MoveInProgress func(worktree string) (wayForward string, standing bool, err error)`, which `hubDepsAs` fills from `fabricengine.MoveInProgress` over that worktree's weft.
`adopt`, after its argument checks and `namedEntries`, and before its first step (the origin reconcile), asks it for the prime and for every worktree whose runs it would resume or under which it would add a pair: each named entry's own worktree when present, and each named entry's parent worktree (`ParentOf`, the prime for `""`).
A standing record refuses with the way forward: a change in progress while the record's move lock is held, and `lyx fabric anchor <worktree> --set <B>` once it is free, then the same `lyx adopt` again; so neither `Topology.Add`, the detached first passes, `lyx loom resume` nor the finalize gate suite runs under a moving anchor.
A read error refuses naming it.
A nil `MoveInProgress` reads as no record standing.
`adoptcli/doc.go` names the refusal.

`TestAdopt` gains rows on its fake deps, whose `world.deps` fills `MoveInProgress` from a per-worktree map of standing records: a standing record on the prime, on a named entry's worktree and on its parent each refuse naming `lyx fabric anchor` and `lyx adopt`, with no `Reconcile`, `AddPair` or `ResumeRun` call; a record on an unrelated worktree does not refuse.

**ImpactSummary:** one new deps field with its hub filler and one refusal ahead of the reconcile; the test fake gains the field.
