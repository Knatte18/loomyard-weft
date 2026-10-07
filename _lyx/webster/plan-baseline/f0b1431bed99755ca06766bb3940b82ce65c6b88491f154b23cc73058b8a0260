# Card 6 — batten-watched-marker

**Edit:**
- `internal/battenshed/deps.go`
- `internal/battenshed/seamchild.go`
- `internal/battenshed/seamchild_test.go`
- `internal/battenshed/innerrun.go`
- `internal/battenshed/innerrun_test.go`
- `internal/battenshed/doc.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_test.go`
- `internal/battencli/wire_seams_test.go`

**Uses:**
- `internal/shedrun/paths.go`
- `internal/hubgeom/parent.go`
- `internal/orchcli/paths.go`

**Intent:**
D6, batten side: batten writes the `batten-watched` marker (card 3's `shedrun.BattenWatchedMarker` of the task worktree's `SelfRunID` run) holding its own pid while its notices have a destination that is also the driver's parent, and removes it otherwise.

`battenshed` gains one seam, `MarkWatched func(ctx context.Context) (held bool, err error)`, on both `SeedChildDeps` and `InnerRunDeps`: it writes or removes the marker and reports whether this batten now holds it.
Seed-Child calls it after the seed is written and committed, before returning `Done`, so the child's first driver launch already renders the watched rule; an error is a `logger.Warn` and never changes the verdict, like the push.
InnerRun calls it at the start of every `Call` and once per `notice_probe_s` inside card 5's wait, keeps the last answer in the producer as "holds the watched marker" for card 7's awaiting rule, and reads an error as not held with one `Warn` per change of answer.
A nil `MarkWatched` resolves to one reporting not held, in both constructors.

`battencli/wire.go` wires one shared closure for both: the marker is written only when all hold, and removed (absent is fine) otherwise:
- the run's parent, `hubgeom.ResolveParent` on the task worktree's location, names the prime worktree batten runs in (`Parent.Worktree` equals the prime location's worktree name); a pair created from another worktree, or with no resolvable parent, gets no marker;
- the prime orch state (`orchengine.LoadState(orchcli.PrimePaths(location))`) records a strand.
The marker's directory is created as needed, and the write is atomic (temp file and rename) so the render never reads a torn pid.
An absent task worktree is an error the seam returns.

Bound, stated in `internal/battenshed/doc.go`: only the driver's "run stopped" message is removed, and only where a live batten with a notice destination replaces it at render time; a driver launched unwatched and adopted later keeps today's rule (a duplicate, never a loss), and one launched watched keeps the silent rule whatever happens to batten afterwards, so a batten that dies after the render, or an orch strand that disappears after it, leaves the run's stops unannounced until a batten runs again, which then sends the notices not yet sent.

Tests:
- `seamchild_test.go`: Seed-Child calls `MarkWatched` after the commit, and a failing one still returns `Done`;
- `innerrun_test.go`: the call records the seam's answer, re-asks it no more than once per `notice_probe_s` inside the wait, and a nil seam reads as not held;
- `wire_test.go`/`wire_seams_test.go`: over a hub fixture, the marker is written with batten's pid for a pair created from the prime with an orch strand recorded, and removed for a pair whose origin names another worktree and when no orch strand is recorded.

**ImpactSummary:** two seam structs gain one optional field each and both producers call it; `battencli`'s wiring gains one closure; nothing outside batten reads the marker except card 3's render.
