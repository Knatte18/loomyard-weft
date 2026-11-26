```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

**[MEDIUM] D6, "a failed `Pool.Holders` read such as one that meets a half-written holder record"**
- **Problem:** A holder record is never half-written, so the example names a failure that cannot occur.
- **Evidence:**
  - `tryAcquireSlot` writes the record through `fsx.AtomicWriteBytes` (`internal/gateslot/pool.go:324-326`).
  - `AtomicWriteBytes` writes a temp file, `os.CreateTemp(dir, ".tmp-")`, and then renames it (`internal/fsx/fsx.go:57`).
  - A record that is held but not yet written is skipped on `os.ErrNotExist` (`pool.go:354-355`).
  - `Holders` does fail on a `lockIsHeld` error, a read error or a YAML parse error (`pool.go:346-348`, `:357-362`), so the failure path the fix added is real. Only the example is false.
- **Suggested change:** name a real cause instead, such as an unreadable or unparsable held record, or a lock probe that fails.

**[LOW] D3 rationale, "the landing notice decides and drops its memory at the Worktree-Teardown row, a tick before the run derives as landed"**
- **Problem:** "A tick before" does not always hold.
- **Evidence:**
  - `PastLand` is also true once batten reads done (`internal/battenrecipe/land.go:28`).
  - If the run passes Worktree-Teardown and reaches done between two ticks, `rememberRuns` decides the notice on the same tick the run first derives as landed (`internal/runfsm/state.go:71-76`).
  - The decision itself is unaffected, because the skip rests on the watch's own copy. Only the rationale overstates the timing.
- **Suggested change:** "at the latest on the tick the run first derives as landed, often a tick before".

Claims checked and found to hold:
- `d.listedRuns` against `d.runs` (`passes.go:181-188`).
- The scheduling gate (`loop.go:708-729`).
- `Halted` reads only the child's state, and a blocked Land row derives as landing (`state.go:87-98`).
- The cancel's own notice runs after `clearRunPhase` on the remove path (`cancel.go:276-279`).
- Each slot's `Started` is stamped separately (`pool.go:322`).
- `yaml.v3` `Unmarshal` with no `KnownFields` (`pool.go:17`, `:360-361`).
- `nanotime1` uses clock id 1, `CLOCK_MONOTONIC` (`sys_linux_amd64.s:223`, `:270`).
- A pause lifting over a blocked child gives a stop reading with a changed state.
- "Both outcomes derive landed" once batten reads done.
- A run first read after its Land row can get both lines.

Tool calls:
1. `grep`/`sed` over `internal/gateslot/pool.go`, `internal/fsx/fsx.go` and `internal/battencli/cancel.go` plus `cancelcmd.go`, for the YAML import, the atomic write and the cancel notice path. My context did not show these spans.
2. `grep` for the yaml import and `KnownFields` in `internal/gateslot`. The first call's pattern missed the import line.
