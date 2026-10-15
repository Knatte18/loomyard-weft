# creel: inject --interrupt stalls behind the prompt Claude restores into the input box

`creel-interrupt-restored-prompt` · Note · bug, creel, shuttle, claude

Imported from [issue #659](https://github.com/Knatte18/loomyard/issues/659).

Found by the creel stage 1 audit (board note `creel-test`), in a sandbox hub, with `lyx` built from `main` at e131c7625.

## What was seen

`lyx creel inject --interrupt` does not deliver its line in the common case.
When Claude Code is interrupted early in a turn (here 4 to 9 seconds in, still thinking), it puts the interrupted prompt back into the input box.
Creel's next idle probe then reads a draft in the box and answers busy, so the head-queued inject waits until someone clears the box or its TTL expires.
`lyx creel interrupt` leaves the strand in the same state: every later reminder and inject is held behind the restored prompt.

## Steps

1. In a sandbox hub, a Claude strand `ctest:eve` in its own window, idle.
2. `lyx creel inject --wait ctest:eve "Without using any tool, write a 700-word essay about mountains."` (outcome injected).
3. Nine seconds later: `lyx creel inject --interrupt --wait --timeout 40s ctest:eve "Reply STOPPED2."`.
4. The journal shows `interrupted` and the inject `posted`; the call times out (exit 4).
5. The pane shows the essay prompt back in the input box; nothing more is typed.
6. Clearing the box by hand (C-u) lets the queued inject go at the next probe.

## Expected

`inject --interrupt` stops the turn and then types its line, as its help example `"stop and run the tests" --interrupt --wait` promises.
For example, the guarded interrupt (or the inject path after it) clears a box that holds exactly the interrupted prompt, or creel treats that restored text as its own to clear before typing.
