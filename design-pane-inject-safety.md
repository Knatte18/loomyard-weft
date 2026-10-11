# Pane injection never merges with the operator draft, and batten survives a draft

`pane-inject-safety` · Task · bug, shuttle, batten, orch, reed · running · Batchifier

Two faults around lyx typing into a Claude pane, merged into one task (operator, 2026-10-11).

## Injected text merges with the operator's draft (was `pane-inject-race`)

### Seen (2026-10-11, operator)

Text lyx injects into a Claude pane (batten notices, resume pointers, driver resume lines, `/clear`, `/compact`, `/reload-plugins`) regularly merges with what the operator is typing, or wipes it.
The draft guard checks the input box once, before typing, and the send then spends about a second typing into the pane (Escape, 300 ms sleep, text, settle reads, Enter) with nothing stopping the operator from typing in between.
The first Enter of a verified send (`confirmSubmitted`, `internal/shuttleengine/send.go:346`) is never checked against the box, so a merged draft is submitted.
`boxHoldsSentText` (`internal/shuttleengine/run.go`) only matches the leading 48 characters, so a merged box reads as "holds the sent text" and gets further Enters.
`ClearSession`, `CompactSession` and `ReloadPlugins` (`internal/shuttleengine/session.go`) type the command and a blind Enter on the strength of the caller's earlier probe, so `/clear` can land on an operator draft.
`judgeReadiness` (`internal/shuttleengine/readiness.go:83`) reads ready when no box is found, so a menu or picker on screen passes the guard, and the leading Escape then closes it.
The leading Escape can complete a double-Esc that clears the operator's draft, or interrupt a turn the operator just started.

### Wanted

A check that refuses the Enter after the text merged comes too late: the operator's draft is already spoiled.
So prevent the merge first, and keep the Enter check only as a last guard (operator, 2026-10-11):

1. **Defer while the operator types.** Hold an injection while the attached client's `#{client_activity}` is under a few seconds old and the target pane is its active pane; the injection waits in a queue and is retried, never dropped.
2. **Send text and Enter atomically.** `load-buffer` + `paste-buffer -p -d` + `send-keys Enter` as one tmux command list, and drop the leading Escape and the 300 ms settle, so the window between check and submit shrinks from about a second to the capture-to-write gap.
   Needs a live check that Claude Code treats a paste-end followed by Enter as a submit.
3. **Last guard: check the box right before Enter.** Press Enter only when the box holds exactly the sent text (or the paste placeholder).
   On a mismatch, send no Enter, never clear or edit the box, and requeue the injection until the box reads empty again (the operator sent or cleared the draft).
4. Route `/clear`, `/compact`, `/reload-plugins`, color and session-rename through the same path, and treat an unreadable box as not idle in `judgeReadiness`.

Later, not in this task: file-based notice delivery to the orch through a Stop hook, so fewer notices are typed at all.

## A draft in the driver pane kills batten, and the pair is never torn down (was `batten-draft-teardown`)

### Seen (2026-10-11, gate-test-hygiene, PR #537)

After `lyx loom approve` at PR-Gate, batten's Run-Shed tried to resume the parked driver and failed with `shuttle: the session is not idle ... the input box holds a draft: "Tok"`.
`internal/battenshed/innerrun.go:816` treats any resume error other than `ErrChildNotParked` as fatal, so batten went `failed` and exited.
The orch then drove the loom run to done by hand with `lyx loom start`.
Nobody resumed batten, so Worktree-Teardown never ran: the pair, its empty reed session and its weft branch stayed, and no `archive/<slug>/<tip>` tag was pushed.
`lyx batten run <slug>` afterwards closes it: Run-Shed sees the done child and moves on to Worktree-Teardown.

### Wanted

- A "session not idle / draft" resume error is not fatal: batten retries it the way `ErrChildNotParked` is retried (with the queue above, the resume waits for the box to clear), or halts `blocked` with the remedy "clear the draft, then `lyx batten run <slug>`".
- A batten run that is `failed` while its child loom run is `done` is flagged, naming `lyx batten run <slug>` as the step that closes it.
- The orch's after-landing routine checks that `lyx batten status <slug>` is done, and otherwise runs `lyx batten run <slug>`.
