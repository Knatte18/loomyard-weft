# Webster counts Master's turn end past a still-running background shell

`webster-background-shell-wait` · Note · bug, webster

Imported from [issue #397](https://github.com/Knatte18/loomyard/issues/397).

## What happened

`lyx webster` wrote a `webster-background-shell` note titled "Master's wait counted a turn end past a background shell".
It lists a background shell, a polling `until grep -q '^done$' <log>; do sleep 5; done` loop, that had run past `background_shell_wait_min` (10 minutes) while Master's turn end was being judged.
The note says the shell may still be running in the session.

## Why it is abnormal

The note records that Master's turn end was counted although a background shell was still alive past the wait limit.
It is unclear whether the turn end should have been held until the shell finished, or whether the shell should have been stopped.
A shell left running across a turn end can outlive its purpose or race with the next step, and the note gives the operator no outcome: whether the shell was killed, whether the wait gave up, and what the run did next.

## What to do

Decide the intended behavior when `background_shell_wait_min` elapses with a shell still running: keep waiting, kill the shell, or escalate.
Make the note state which of these happened.

Provenance: task `test-suite-measure`; notes: `webster-background-shell.md`.
