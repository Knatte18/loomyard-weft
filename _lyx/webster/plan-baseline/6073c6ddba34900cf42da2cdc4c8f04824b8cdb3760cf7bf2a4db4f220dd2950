# Card 5 — gitexec-network-hold

**Edit:**
- `internal/gitexec#runCore`
- `internal/gitexec_test#TestRun_RemoteBounds`
- `internal/gitexec/doc.go`

**Intent:**
Hold every network git call while a pause covers the calling process (D4's "network git in `internal/fabricengine`: push, fetch, pull, a landing's remote calls").
`runCore` already tells a remote subcommand (`remoteSubcommands`: push, fetch, pull, ls-remote, clone) from a local one; for a remote one it now calls `done := pauserecord.Network(subcommand(args))` before it creates the remote deadline's context, and defers `done`.
So the wait counts against no deadline, and a held push starts its ten-minute bound only when it runs.

This is the one runner under every fabricengine remote call, `gitexec.Run` at its own call sites and gitrepo's `Push`, `Fetch` and `Pull`, which fabricengine drives, so the hold sits once beneath fabricengine rather than at each of its many remote sites or at any of its callers (`PATTERN-fabric-git`); Shared Decision `network-hold-chokepoints` records the choice.
A local subcommand is never held, and with no gate installed (every test binary and every process before card 23's root hook) `Network` is a no-op.

Bound: it only delays a remote git call at its start, never fails one, and lets a remote call of a run whose pair holds a parent lock through under a per-run entry through the gate's `HoldsParentLock`.

Test: `TestRun_RemoteBounds` gains a subtest, run before the deadline subtests change the kill reporter: it installs a `pauserecord.Gate` over a temp record holding a live hub entry, with a `Sleep` that signals each wake, runs `ls-remote ext::<script that writes a marker file and exits 0>` in a goroutine, asserts the marker file is absent after the gate's first wake, resumes the entry through `pauserecord.Update`, and asserts the command then returns with the marker written; a local `git -c alias.ok=!true ok` run under the same live entry returns at once.
`doc.go` names the hold in the remote-subcommand paragraph.

**ImpactSummary:** One extra call in runCore's remote branch; every remote git caller in the module gains the hold, and nothing changes while no gate is installed.
