# A producer session starts without its prompt delivered, or beside a stale one

`producer-session-handoff` · Note · bug, reed, shuttle, loom

Imported from [issue #395](https://github.com/Knatte18/loomyard/issues/395).

reed's `SendText` sometimes types a prompt into a Claude Code pane without submitting it, and the session sits with the text in its input box until someone presses Enter.

Seen 2026-10-05 at `lyx orch start --handoff …`: the launch pointer (`Read …/prompt.md in full first; …`, two wrapped lines) stayed unsent; the operator pressed Enter by hand.
The short skill loads before it (`/scribe:prose`, `/ly:board`) were submitted.

`SendText` (`internal/reedengine/io.go`) runs `send-keys -l <text>` and then `send-keys Enter` back to back.
Claude Code's input treats a fast burst of characters as a paste, and an Enter that arrives inside the burst becomes part of it instead of a submit.
The longer the text and the busier the machine, the likelier the race.

Fix options:

1. Send the text as a bracketed paste (`load-buffer` + `paste-buffer -p`), so the TUI knows where the paste ends, then Enter.
2. After Enter, confirm the input box emptied (capture the pane, or wait for the transcript's user record) and resend Enter once if not.

Option 2 also covers other causes of a lost Enter; the two combine.
Every typed delivery (shuttle launch pointers and skill loads, orch watcher reloads, batten notices, `SendMessage`-style relays through reed) goes through this path.


## From issue #399

Imported from [issue #399](https://github.com/Knatte18/loomyard/issues/399).

`reflectHalt` (`internal/loomcli/halt.go`) writes `loom-halt.md` and at once runs `reflectFriction(false)`, which spawns a reflection agent session on every halt.
That includes a `blocked` halt classified `escalation-to-human`, where the producer's own session is still live and holds the open question.

Seen 2026-10-06 on `test-suite-prune-core`: Plan-Write halted `blocked` with "agent is asking a question".
While the plan session stood waiting, the run spawned a friction-reflection session; its report said only "an escalation, a process signal and not a lyx defect".
To the operator this looked like the run moving on while its producer hung.

Expected: an `escalation-to-human` halt writes the halt note but spawns no reflection.
The terminal Friction-Reflect row reads the note anyway.
