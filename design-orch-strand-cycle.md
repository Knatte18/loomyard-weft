# Orch lifecycle: dead strands count as dead, the watcher restarts, cycle requests need no reply and can be withdrawn

`orch-strand-cycle` · Task · bug, enhancement, orch, reed, shuttle, creel · running · Discussion-Burler-Fix

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

## Folded in: dead strand names (operator, 2026-10-22)

Note `reclaim-dead-strand-names` (issues #799, #802, #811) is folded into this task, since it is the same area: a dead strand, of any role, holds its reed name after a reboot or respawn, so a resumed row or a Burler-fix row fails on the name clash.
A respawn reclaims a dead strand's name and registration.
When it lands, the hub orch rules section "Dead strands after a reboot (#799)" is removed.
The `loop-exited` halt note #811 also names is out of scope here: task `loop-exit-halt-names-its-cause` takes it.
Its body follows.

### Seen (orch, 2026-10-21, OSL-1046; issue #799)

After a machine restart, both in-flight runs were resumed from the prime with `lyx batten run <slug> --window`.
Reed still listed the strands from before the restart as dead (`live: false`), holding their names.

- `test-tiers`: Plan-Burler-Fix re-spawned its fixer as `ly:test-tiers:plan-burler-fix-2`, and the row failed with `seatengine: strand "ly:test-tiers:plan-burler-fix" started as "ly:test-tiers:plan-burler-fix-2", because another strand holds that name; way forward: run "lyx reed remove --name ly:test-tiers:plan-burler-fix", then re-step the row`.
- `tuned-defaults-template`: Merge-In's conflict agent came up as `conflict-2` beside the dead `conflict` and carried on.

Workaround: `lyx reed remove --name <dead strand>` in the task worktree, then `lyx loom resume`.

Expected: a strand whose pane is gone, such as every strand after a reboot, does not hold its name: a resume or a re-spawn reclaims the name of a dead strand without a manual `lyx reed remove`, and the row does not fail.

### Seen in the quarry hub (issue #802)

The machine slept while task `csharp-alphabet` was at the Webster producer, which killed the Webster master strand `qry:csharp-alphabet:webster` and its batch fork.
After `lyx batten run csharp-alphabet --window` and `lyx loom resume`, Webster respawned its master as `qry:csharp-alphabet:webster-2`, since the dead strand's registration (`live: false`, no pane) stayed in `lyx reed list` and held the name.
The orch removed the dead registration by hand with `lyx reed remove <guid>` from the task worktree.

Expected: when a strand is respawned for a role whose previous strand is dead (not live, no pane), reed or the respawning caller drops the dead registration first, and the new strand keeps the canonical name, so a lookup by role name cannot hit the dead record.

### Seen at Plan-Burler-Fix (issue #811)

A run halted as `failed` (anomaly `producer-hard-failure`) at `Plan-Burler-Fix`, round 3 attempt 3, after two stuck plan review rounds:

```
shedadapters: Plan-Burler-Fix (burler-fix): round 3 attempt 3: fix: burler: shuttle run: seatengine: strand "ly:<slug>:plan-burler-fix" started as "ly:<slug>:plan-burler-fix-2", because another strand holds that name; way forward: run "lyx reed remove --name ly:<slug>:plan-burler-fix", then re-step the row
```

The earlier fix attempts of the same row had finished (history `stuck`), so their strand should have been released; a leftover strand from a finished attempt turns a routine retry into a hard failure that ends the run.
What to do: have the row's attempt teardown remove its strand, or have `seatengine` reclaim a stale strand of the same name instead of renaming and failing; if a live holder is possible, the retry path runs the `lyx reed remove` itself when the holder is provably dead.

Provenance: tasks `test-tiers`, `csharp-alphabet`; notes `loom-halt.md`, `loom-halt-2.md`.

## Folded in: the watcher re-sends a delivered compact (operator, 2026-10-22)

Imported from [issue #828](https://github.com/Knatte18/loomyard/issues/828).

Seen (loomyard hub orch, Neve, 2026-10-22): at a distill cycle the watcher's /compact inject was delivered and the session compacted, then creel logged a draft stall and sent the inject again twice; the operator interrupted both by hand.
Meanwhile watch.log held "a creel call failed; the cycle is held until creel answers", because creel could not resolve the worktree of a run torn down mid-lookup.

Expected: once a compact inject has been delivered and the session compacted, the watcher does not re-send it; a draft stall after delivery is not read as non-delivery.
A creel strand lookup that meets a worktree removed mid-teardown skips it instead of failing the call.


