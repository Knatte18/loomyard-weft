# Card 18 — hubgeom-live-orch-walk

**Create:**
- `plan:internal/hubgeom#NearestLiveOrch` -> `func NearestLiveOrch(from *lyxcwd.Location, live func(worktree *lyxcwd.Location) (bool, error)) (Parent, error)`
- `plan:internal/hubgeom#nearestLive` -> `func nearestLive(start string, parentOf func(worktree string) (string, bool, error), live func(worktree string) (bool, error)) (string, error)`
- `plan:internal/hubgeom#TestNearestLive` -> `func TestNearestLive(t *testing.T)`

**Extends:**
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/hubgeom#ResolveParent`
- `internal/hubgeom#decideParent`

**Intent:**
D13's walk, a hubgeom function of its own beside `ResolveParent`, which keeps its answer for every reader that needs the parent pair itself (Finalize's parent-records commit among them).
`NearestLiveOrch` tries the worktree `from` names, then walks upward through origin records (each worktree's `ParentWorktree`), returning the nearest worktree whose orch strand is live by `live`, and the prime whatever its state when none is.
A caller acting on a run's pair passes the pair's parent worktree (card 20), and batten passes the worktree its run's pair forks from when that pair is absent (card 25).
The answer is a `Parent` whose `Name` is that worktree's orch agent name (`decideParent`'s formatting: `<shortname>:orch` for the prime, `<shortname>:<slug>:orch` for a beam) and whose `Worktree` names it, empty for the prime.
The predicate is told, since the orch's state paths and reed belong to `orchcli`, which imports hubgeom; card 19's `orchcli.OrchLive` is the production predicate.
A worktree with no origin record, an empty `ParentWorktree` or a parent worktree that is gone ends the walk at the prime; a cycle ends at the prime after visiting each worktree once; a `live` error is returned.
`nearestLive` is the walk over told reads, so `TestNearestLive` needs no filesystem: rows for a live start, a stopped beam under a live parent beam, no live orch (the prime), a gone parent worktree (the prime), a cycle (the prime) and a predicate error.

**ImpactSummary:** Adds one resolver beside ResolveParent; no existing caller changes.
