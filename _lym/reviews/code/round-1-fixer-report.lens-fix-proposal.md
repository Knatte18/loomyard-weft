```yaml
lens: fix-proposal
outcome: reported
```

No findings against the reviewer's proposals.

I tried to build a failing case for each proposal I was handed:

- **F1:** The stat gate only adds the pins path to the commit when the file exists. A legacy run with no pins file adds nothing, so git cannot refuse an empty pathspec. A re-entry over an already-committed pins file matches a tracked path, so that commit is also valid. I found no breaking case.
- **F2:** The reworded notice still contains `"lym stencil diff <name>"`, which the test asserts. The other `lym stencil diff` mentions I grepped do not claim it shows changes since the run started. Those are `friction.go:109`, `burlerengine/prompt.go:269`, `reconcile.go:332` and the `stencilhistory.go` header. No stale twin is left.
- **F5, F6:** The parked-wake row and the failed-post row both assert the envelope is ok and `stencils_changed` names the edited stencil. A failed post still counts as one notice, so `assertParentNotices` holds.
- **F7:** A directory in place of the board copy makes `os.ReadFile` fail with a non-`ErrNotExist` error. That reaches the `%q`-wrapped message naming `"loom-two"`. The ordering row only passes if `ChangedPins` sorts, since the map iteration order is otherwise random.
- **F8:** `mustSpawn || parkedLive` covers every branch that puts a driver to work. A held driver lock with no strand and no marker is correctly reported as false. The only cost is that a stencil change goes unreported on a no-op start, which is what the review asked for.
- **F9:** `succeedAfterWake` releases the bootstrap lock before the post. After the release, a concurrent start could enter while the notice is still being posted. That is harmless, because the post reads only the pins and the installed stencils.

One note on the own-change F4, not a proposal flaw. No test fails if the `pinsStatErr == nil` gate in `sharedbootstrap.go:183` is dropped or inverted. The pathspec row only pins `bootstrapCommitPaths`' shape, and the re-entry commit stays untested against real git.

Calls made:
1. Grepped `stencil diff` and `StencilPinsFile` across non-test sources and docs, to find stale twins of the reworded F2 promise and other users of the pins path. The context held only the files the findings named.
