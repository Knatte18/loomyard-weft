# Orch lifecycle: dead strands count as dead, the watcher restarts, cycle requests need no reply and can be withdrawn

`orch-strand-cycle` · Task · bug, enhancement, orch, reed

Depends on: [`run-process-ownership`](design-run-process-ownership.md)

Three orch lifecycle fixes in one run: a dead strand counts as dead, a dead watcher restarts, and the watcher's context-cycle request needs no reply and can be withdrawn.

## 1. A strand whose engine has exited counts as dead

Seen on Linux (tmux 3.4) in the loomyard hub's prime ([issue #746](https://github.com/Knatte18/loomyard/issues/746)).
The operator closed the orch Claude session by mistake and then talked to the same session id outside reed.
`lyx orch status` afterwards reported `strand_live: true` with `watcher_live: false`, while the orch strand's pane (`%3`) held only a `bash` shell, under a session and window named `0` and `bash`, not the prime's `loomyard:orch`.
`lyx orch start --continue` and `--adopt` are refused while the strand is live, so getting back needed `lyx orch stop` first.

Today reed counts a strand live when its recorded pane id is listed by tmux and not dead (`internal/reedengine/lifecycle.go`, the status path's `aliveIDSet`); the process in the pane is never checked.

### Operator decision (2026-10-21)

A strand is live only when all three hold:

1. its pane exists and is not dead, as today;
2. the pane sits in the reed session and window the strand was launched into;
3. the pane's foreground process is the engine (`claude`), not a shell.

When any fails, the strand is dead: `lyx orch status` shows `strand_live: false`, and `lyx orch start --continue` resumes in one step with no `lyx orch stop` first.
The check lives in reed and holds for every strand, not only orch's, so webster's `StrandLive` gets it too.

Reed never adopts a pane outside its own session.
`start --continue` and `--adopt` always launch orch in the prime's reed window (`<hub>:orch`); a stale pane elsewhere is left alone, and status only names it.

## 2. A dead watcher restarts

### Operator decision (2026-10-21)

A dead orch watcher is restarted, not only reported: at `lyx orch status` and at every `lyx orch start`, and the restart is logged so it shows.

## 3. The watcher sees the orch note is written, with no reply

Seen (2026-10-18): at a context cycle the orch watcher posts a request to write the orch note, and waits for the orch to answer `lyx creel post --reply <id> "written <path>"` before it compacts or clears the session.
The reply costs the orch one more tool call per cycle.
The watcher already reads the session's turn ends, since a soft cycle waits for the newest turn end to be quiet.

### Operator decision (2026-10-18)

- The watcher sees by itself that the note is written: the orch's turn has ended and the named note file exists and is not empty; it then cycles with no reply.
- A `defer` reply to a soft request stays, since only the orch knows that work a cycle would lose is in flight.

## 4. A pending cycle request can be withdrawn

Seen (orch, 2026-10-21): the orch ran `lyx orch distill` while a hub pause stood.
The watcher starts no context cycle under a hub pause (`internal/orchengine/doc.go`), so the request sat pending until the resume.
The operator wanted it withdrawn, and no verb does that: the orch deleted the marker `.lyx/orch/cycle-request` by hand.

Wanted:

- a verb that withdraws a pending `refresh` or `distill` request, refusing once the cycle has started;
- `lyx orch status` shows a pending request;
- `distill` and `refresh` say when a hub pause holds the request back.
