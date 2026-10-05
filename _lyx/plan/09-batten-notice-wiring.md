# Card 9 — batten-notice-wiring

**Edit:**
- `internal/battencli/wire.go`
- `internal/battencli/wire_seams_test.go`

**Uses:**
- `internal/orchcli/paths.go`
- `internal/orchengine/notice.go`

**Intent:**
Wire the Run-Shed row's notice seams to the prime's orch queue (D13).

In `battenCLI.wire`, `InnerRun.Notify` calls `orchengine.QueueNotice` with `orchcli.PrimePaths(location)` for the prime `location` batten runs from and the current time; a notice it does not queue (no orch strand recorded) is logged there.
`InnerRun.AttachDir` resolves the task worktree's anchor path through `taskWorktreeLocation`, on Call like the other seams.
This assumes the orch's batten runs are started from the prime, so the prime's orch is their parent.

Tests in `wire_seams_test.go`: with an orch state recording a strand, a notice lands as one file under the prime's `.lyx/orch/notices/`; with none recorded, nothing is written; `AttachDir` names the task worktree.
Follow the existing seam tests' fixture shape and tier.

**ImpactSummary:** Wires two new InnerRun seams in battencli, adding an import of orchcli and orchengine; no other row's wiring changes.
