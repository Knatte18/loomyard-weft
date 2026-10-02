# Card 2 — parent-resolver

**Create:**
- `plan:internal/hubgeom#Parent` -> `type Parent struct`
- `plan:internal/hubgeom#ResolveParent` -> `func ResolveParent(l *lyxcwd.Location) (Parent, error)`
- `internal/hubgeom/parent.go`
- `internal/hubgeom/parent_test.go`

**Edit:**
- `internal/hubgeom/hubgeom.go`
- `internal/hubgeom/doc.go`
- `internal/hubgeom/reedgeom_integration_test.go`

**Uses:**
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#ReadShortname`
- `internal/shedrun#ReadSeed`
- `internal/agentname#Format`
- `internal/agentname#RoleOrch`

**Intent:**
Add the one parent resolver Decision 3 names, in a new `internal/hubgeom/parent.go`, so `parentreview` and `reedengine` stay told-geometry.

`Parent` carries `Name`, the parent's full agent name or empty for no parent, and `Worktree`, the parent worktree's name or empty when the name came from the legacy seed.
`ResolveParent(l)` reads the pair's origin record through `fabricengine.ReadOriginFor(l, l.WorktreeName)` (it creates nothing on disk), the hub shortname through `fabricengine.ReadShortname(fabricengine.BoardDir(l.HubPath))`, and the default run's seed through `shedrun.ReadSeed(l, shedrun.SelfRunID)`, then answers:

- origin record with `ParentWorktree` set: the orch role's full name in that worktree, formed by `agentname.Format(shortname, slug, agentname.RoleOrch)`, where `slug` is empty when `isPrimeWorktree(filepath.Join(l.HubPath, ParentWorktree))` reports the prime and is `ParentWorktree` otherwise; `Worktree` is `ParentWorktree`;
- no `ParentWorktree` (whether or not an origin record exists) and a non-empty legacy seed `Parent`: that name verbatim, `Worktree` empty;
- otherwise: no parent.

No string concatenation forms a name; `agentname.Format` is the only former.
The resolver checks no liveness: the name is returned even when no orch strand is live.
A hub with no recorded shortname yields no parent and logs one `logger.Warn`, since no name can be formed and reed already refuses spawns without a shortname.
Origin, seed and prime-check read errors are returned.

Split the decision into a pure helper over already-read values (shortname, origin and found flag, legacy seed parent, and whether the parent worktree is the prime) so the table is testable with no filesystem.

`reedGeometry` replaces its direct seed read with `ResolveParent` for a task worktree: `ParentName` becomes `Parent.Name`, a resolver error logs the existing Warn and leaves `ParentName` empty, and the prime still gets no parent.
This is what gives every strand in a pair `LYX_PARENT` equal to the prime's orch name whoever ran `lyx batten run`.
Update `ReedGeometry`'s doc comment and `doc.go`, which today say the parent comes from the default run's seed.

Tests: `parent_test.go` is a table over the pure helper covering the acceptance rows — prime parent gives `<shortname>:orch`; pair parent gives `<shortname>:<slug>:orch`; no origin record and no legacy seed gives none; origin without `parent_worktree` plus a legacy seed name gives that name; both absent gives none; no shortname gives none.
In `reedgeom_integration_test.go`, a pair created by the hub fixture from the prime now carries `ParentName` `<shortname>:orch` before any seed is written (replace the "empty before seeding" assertion), and an origin record rewritten without `parent_worktree` plus a seed carrying a legacy parent yields that legacy name.

**ImpactSummary:** Changes where reed's `ParentName` comes from for a task worktree (origin record first, legacy seed second); the prime and standalone geometry are unchanged.

**Commit:** `2: hubgeom resolves a pair's parent agent name from its origin record`
