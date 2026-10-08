# Card 10 — Revive a restarted server's sessions in spawn order

**Edit:**
- `internal/reedengine/geometry.go`
- `internal/reedengine/lifecycle.go`
- `internal/reedengine/strand.go`
- `internal/reedengine/lifecycle_test.go`
- `internal/reedengine/doc.go`
- `internal/hubgeom/hubgeom.go`
- `internal/hubgeom/doc.go`

**Create:**
- `internal/reedengine/revive.go`
- `internal/reedengine/revive_test.go`
- `internal/reedengine/revive_integration_test.go`
- `internal/hubgeom/spawnorder.go`
- `internal/hubgeom/spawnorder_test.go`

**Intent:**
Keep D5's and D10's session-id order across a server restart (D12 and the booter-skip addendum): when a worktree revives its recorded session, every earlier worktree in spawn order whose own session needs reviving is brought up first, so session ids follow spawn order again.

Told list (`geometry.go`): `Geometry` gains `SpawnOrder func() ([]ReviveEntry, error)`, nil in standalone mode, and `ReviveEntry{Worktree string; Revive func() (revived bool, err error)}`.
The function is lazy so building an engine costs nothing; it is called only when a revival is due.
`reedengine` sorts nothing, derives no path and reads no loom or fabric record (`PATTERN-told-geometry`); the booter's predecessors are the entries before the one whose `Worktree` equals `Geometry.WorktreeName`, and a list without the booter has none.

Revival predicate, one test evaluated by a worktree only on its own state under its own op lock: its `reed.json` records a session and that session is not live.

Engine (`revive.go`, `lifecycle.go`):

- `ensureServerAndSessionLocked` takes a skip option. At the point where it would create the booter's session (no session up, after the foreign-session refusal, before the stale-socket reap and the spawn), when the option is unset, `SpawnOrder` is non-nil and the predicate holds, it returns a sentinel and creates nothing.
- The steps that reach it (`Up`, `EnsureSession`, `Resume`, `AddStrandUnless`) run through one helper: run the step under the op lock without the skip; on the sentinel, release the lock, call `SpawnOrder`, run each predecessor's `Revive` in list order, sequentially and outside the booter's lock, then re-take the lock and re-run the whole step from its start with the skip set, so a session another booter created meanwhile is used as found and the trigger is evaluated once.
  No two op locks are ever held at once.
- `(e *Engine) Revive() (bool, error)`, exported for the teller: under this engine's own op lock, when the predicate holds it runs this worktree's ordinary `up` (session and Selvage, no strand relaunch) with the skip set and reports true; otherwise it reports false and creates nothing.

Bound: the step runs only for a revival of the booter's own recorded session, never on a first boot; it brings up only earlier worktrees for which the predicate holds (a `down` removes that state, so a downed worktree is never revived); it starts no strand; it never recurses, since every nested up carries the skip; an up or a `SpawnOrder` call that fails is logged via `logger.Warn` and skipped, and the boot goes on.
It does not reorder a later-position session that is still live, since session ids cannot change without killing live sessions.

Teller (`internal/hubgeom/spawnorder.go`): `ReedGeometry` sets `SpawnOrder` to a closure over the resolved location that lists the hub's worktrees with `fabricengine.List` (skipping prunable entries), resolves each with `lyxcwd.ResolveWorktree` (an unresolvable one is skipped with a warning), and orders them: the prime first, then each pair by its run's start time, read as the modification time of the pair's own run seed (`shedrun.SeedFile(loc, shedrun.SelfRunID)`, written once when the run is seeded), then pairs with no readable seed by worktree name.
Each entry's `Revive` builds an engine from that worktree's own `reed.yaml` (`reedengine.LoadConfig` at its anchor) and its own `ReedGeometry`, and calls `Revive`.
A pure `sortSpawnOrder` carries the ordering rule.

Tests:
`spawnorder_test.go` (untagged) is one table over `sortSpawnOrder` (prime first whatever its time, dated pairs ascending, undated after them by name, ties by name) plus a row reading the seed time from a temp-dir seed file and one with the seed absent.
`revive_test.go` (fake tmux) pins the trigger: first boot, live session, `SpawnOrder` nil and the skip set each create without reviving; a recorded, not-live session returns the sentinel before any spawn; `Revive` reports false without the predicate.
`lifecycle_test.go` asserts the helper's sequence with a told list of fake revive functions: predecessors in order, the booter's own entry and later entries never called, a failing predecessor skipped, the step re-run once with the skip.
`revive_integration_test.go` (`//go:build tmux`, `tmuxkit.Main`) boots three engines on one private socket with a told list of their real `Revive` functions, kills the server, and asserts: booting the last brings the sessions back with ids in list order, the booter's own at its position; a second booter starting after the first has brought the server up still fills the earlier gaps first; a predecessor whose `Revive` fails is skipped and the boot completes; with a later-position session still live, the booter's session is created after it and the live session is untouched; a first boot and a downed worktree trigger no revival.

Docs: `reedengine` `doc.go` gains the revival section (predicate, lock sequence, bound); `hubgeom` `doc.go` names the spawn-order teller and its start-time source.

**ImpactSummary:** every session-creating reed op gains a revival check before spawning; Geometry gains one optional field set only by the hub-mode teller.
