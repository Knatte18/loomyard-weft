# A draft in the driver pane kills batten, and the pair is never torn down

`batten-draft-teardown` · Note · bug, batten

## Seen (2026-10-11, gate-test-hygiene, PR #537)

After `lyx loom approve` at PR-Gate, batten's Run-Shed tried to resume the parked driver and failed with `shuttle: the session is not idle ... the input box holds a draft: "Tok"`.
`internal/battenshed/innerrun.go:816` treats any resume error other than `ErrChildNotParked` as fatal, so batten went `failed` and exited.
The orch then drove the loom run to done by hand with `lyx loom start`.
Nobody resumed batten, so Worktree-Teardown never ran: the pair, its empty reed session and its weft branch stayed, and no `archive/<slug>/<tip>` tag was pushed.
`lyx batten run <slug>` afterwards closes it: Run-Shed sees the done child and moves on to Worktree-Teardown.

## Idea

- Retry a "session not idle / draft" resume error the way `ErrChildNotParked` is retried, or halt `blocked` with the remedy "clear the draft, then `lyx batten run <slug>`".
- Flag a batten run that is `failed` while its child loom run is `done`, naming `lyx batten run <slug>` as the step that closes it.
- The orch's after-landing routine checks that `lyx batten status <slug>` is done, and otherwise runs `lyx batten run <slug>`.
