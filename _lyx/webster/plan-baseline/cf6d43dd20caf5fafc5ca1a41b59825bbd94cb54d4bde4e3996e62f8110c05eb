# Card 17 — batten revives a halted child's dead driver without resuming the child

**Edit:**
- `internal/battenshed/deps.go`
- `internal/battenshed/innerrun.go`
- `internal/battenshed/innerrun_halted_test.go`
- `internal/battenshed/innerrun_awaiting_test.go`
- `internal/battenshed/doc.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_seams_test.go`
- `docs/overview.md`

**Intent:**
A halted (blocked, paused, failed) or awaiting child whose reed state holds a dead driver strand gets its pair's strands revived through reed resume, which brings back the parked driver with its own session (D6, second and third bullets).
The run itself stays halted and the park marker stays, so a later `lyx loom resume` wakes that driver.

`InnerRunDeps` gains `ReviveStrands func(ctx context.Context) error`; a nil one resolves in `NewInnerRun` to a revive that reports it is not wired, so a caller that wires none (a test building `InnerRunDeps` by hand) sees a failed revive only for a dead driver strand, which its nil `DriverStrand` never reports. `battencli` fills it with the task worktree's reed engine `Resume`, logging the revive and its result (`PATTERN-spawn-observability`).
Reed resume relaunches every non-live, non-hidden strand of the pair's session, so a halted step's gated agent strand and any operator strand come back too, each at its own prompt with no input typed; card 5 drops only an ungated run's finished strand.

In `callHalted` and `callAwaiting`, before the existing wait, the row reads `DriverStrand` (card 16):

- dead: when no revive was attempted in this episode by this batten process, it calls `ReviveStrands`, reads `DriverStrand` again, and records the attempt and its result in a new once-marker under the scratch directory (`<producer>-revived`, holding this process's pid and `ok` or `failed`); a revive that errors, or leaves the driver strand still not live, is warned once and not retried in that episode; a new batten process is a new episode and tries once more;
- retiring: not revived and no warn, since reed resume leaves a live strand alone; it is left to `lyx loom start`, which removes it and spawns a fresh driver;
- none (a Go-driven child, or one whose strand is gone): nothing to revive, no warn; batten never reads the seed's driver to tell these apart (`PATTERN-driver-choice-single-site`);
- live: nothing to do.

A `DriverStrand` read error is warned and skips the revive, never failing the row, since the revive is a convenience on top of the wait.
The revived marker is removed wherever the halt-warned marker is (the child seen out of a halted or awaiting state ends the episode).
Bound: the revive runs at most once per halt episode per batten process, starts no shed step, removes no park marker and sends no resume line.

`haltedWaitReason` replaces its `lyx loom start` text: it names `lyx loom resume` in the task worktree, and also `lyx loom start` in the task worktree once a revive has failed in this episode or when the child has no driver strand in reed state.
The text batten logs and writes to the row's stuck-reason file is that reason; `lyx batten status` does not show it while the child is halted, and no status surface is added.
The awaiting child's hints keep their verbs: `lyx loom approve`/`lyx loom reject`, and `lyx loom start` as the fallback in the hint for a decision already acted on whose driver has ended; none names `lyx loom resume`, since that verb refuses an awaiting run.
The notice line and the notice channel are unchanged.

`innerrun_halted_test.go` pins, per the acceptance criteria: a dead driver strand is revived once, the child's status and park marker are untouched, and a second Call in the same episode does not revive again; a new process (a different pid in the marker) revives again; a retiring strand is not revived and not warned; a failed revive is warned and the reason then also names `lyx loom start`; a halted child with a live driver strand, or a dead one whose revive succeeded, gets a reason naming `lyx loom resume` and not `lyx loom start`; one with no driver strand gets no revive and no warn and a reason naming both.
`innerrun_awaiting_test.go` pins the revive of an awaiting child's dead driver and that its hints are unchanged.
`wire_seams_test.go` adds the new seam to the every-seam-filled list.
`doc.go` replaces the halted-wait sentence: batten revives a dead driver strand once per episode, never resumes the child, and names `lyx loom resume`.
`docs/overview.md`'s batten paragraph on a halted child, which says batten cannot restart the driver and that the reason is on `lyx batten status`, is rewritten per this card.

**ImpactSummary:** Batten's halted and awaiting arms gain a once-per-episode revive and a new halt reason; spawn, done and running arms are unchanged.
