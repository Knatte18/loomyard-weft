# Card 18 — orch-cleanup

**Create:**
- `internal/orchengine/cleanup.go`
- `plan:internal/orchengine#HandoffCleanup` -> `type HandoffCleanup struct`
- `plan:internal/orchengine#CleanupHandoffs` -> `func CleanupHandoffs(p Paths, apply bool) (HandoffCleanup, error)`
- `plan:internal/orchengine#cleanupHandoffs` -> `func cleanupHandoffs(p Paths, apply bool, beforeRemove func()) (HandoffCleanup, error)`
- `internal/orchengine/cleanup_test.go`
- `internal/orchcli/cleanup.go`
- `plan:internal/orchcli#orchCLI.cleanupCmd` -> `func (c *orchCLI) cleanupCmd() *cobra.Command`
- `plan:internal/orchcli#TestCleanup_Envelope` -> `func TestCleanup_Envelope(t *testing.T)`

**Edit:**
- `internal/orchcli#Command`
- `internal/orchcli/doc.go`
- `internal/orchengine/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/orchengine#Paths`
- `internal/orchengine#State`
- `internal/orchengine#NewHandoffPath`
- `internal/lock#AcquireWriteLock`

**Intent:**
Add `lyx orch cleanup [--apply]`, which removes every file under the orch's handoffs directory except the one the persisted `State.LastHandoff` names, with `lyx board prune`'s dry-run shape (D10).

`CleanupHandoffs(p, apply)` is `cleanupHandoffs(p, apply, nil)`, the function the verb and `lyx cleanup` both call.
With `p.Dir` absent it returns at once, removing nothing and creating nothing.
Otherwise it takes the state write lock, `lock.AcquireWriteLock(p.StateLockPath)`, before reading the state and holds it through its last removal, so the watcher, whose every state write takes the same lock, cannot enter handoff-requested in between and lose a handoff it is about to write.
It reads `p.StatePath` with a plain read and JSON decode while holding the lock, never through `LoadState`, which takes the lock itself; an absent file is a zero state in phase idle.
It removes nothing and says why in `HandoffCleanup`'s reason when: the phase is not idle, since a handoff is being written or read in every other phase;
`LastHandoff` is empty;
or the file `filepath.Join(p.HandoffsDir, filepath.Base(LastHandoff))` does not exist.
Otherwise every regular file directly under `p.HandoffsDir` except that one is listed, and removed when `apply` is true, a newer partial handoff an aborted cycle left included; an empty or missing handoffs directory is a no-op.
`beforeRemove`, nil in production, runs after the decision and before the first removal; it is the test seam for the interleaving below.
`HandoffCleanup` carries the kept file, the removed (or, in a dry run, removable) files and the reason.
Bound: the file `LastHandoff` names always survives, so a restart still resumes from it, and a handoff a cycle starts while cleanup runs is never removed; notices, state, locks, the note template and the watch log are untouched.

`cleanupCmd` builds `cleanup` with `Use: "cleanup"`, a lower-case `Short`, audience operator, `cobra.NoArgs` and `--apply`; its `RunE` checks `clihelp.ShouldAbort`, calls `CleanupHandoffs(c.paths, apply)` and prints `{"applied", "kept", "removed", "reason"}` through `output.Ok`, or the error through `output.Err`.
`Command` adds it to the orch subtree, so it inherits the group's prime-only pre-run.

Tests:
`cleanup_test.go` in `orchengine`, untagged and parallel over told paths under `t.TempDir()`, holds one table over the acceptance rows: the `LastHandoff` file kept and the rest removed, a newer partial handoff included; no `LastHandoff`, and a `LastHandoff` naming a missing file, removing nothing; each non-idle phase removing nothing; an empty and a missing handoffs directory; a missing orch directory; and a dry run listing exactly what the apply row removes while removing nothing.
A second test passes a `beforeRemove` that starts the watcher's transition, `updateState` into `PhaseHandoffRequested` followed by writing the new handoff file, in a goroutine, asserts `TryAcquireWriteLock(p.StateLockPath)` fails while cleanup runs, and after cleanup returns waits for the goroutine and asserts the new handoff file exists: a cycle entering handoff-requested during `--apply` loses no handoff.
`TestCleanup_Envelope` in `orchcli`'s `cli_test.go` builds the CLI through the file's existing test constructor and asserts the envelope of a dry run and an apply over a seeded handoffs directory.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/orchengine` and `internal/orchcli` rows grow by their new top-level tests.

Docs in this commit: `internal/orchengine/doc.go`'s "handoffs/: one timestamped handoff file per cycle, all kept" says the handoff `LastHandoff` names is kept and the rest are removed by `lyx orch cleanup`;
`internal/orchcli/doc.go`'s list of verbs names `cleanup`;
`docs/overview.md`'s orch bullet names `cleanup` among the verbs.

**ImpactSummary:** orch gains a cleanup verb over a new engine function; the subtree's pre-run and every existing verb are unchanged.
