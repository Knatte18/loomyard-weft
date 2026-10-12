# Weft records push refused once at Publish, passes on retry

`weft-push-refused-retry` · Note · bug, fabric

On 2026-10-12 the weft records push at Publish was refused once and passed on retry in three runs: `board-field-drop`, `webster-pack-read-set` and `alignment-agent`.

A refusal that clears on retry points at a race on the weft remote, likely another run's push or a board write landing between fetch and push.
The retry hides it today, but each refusal costs a round and a run without retry headroom would halt.

## What to find

- Which ref was refused and why (non-fast-forward or lock), from the Publish trace in each run.
- Whether the refusal comes from concurrent runs writing weft at the same time.
- Whether the push should fetch and rebase before its first attempt, or the retry is the intended path and only needs logging.

## Seen again

2026-10-12 16:04, `lyx batten run intake-bugs-1012` while the orch was writing many board entries in a loop: `gitrepo: push refused, retrying ... ref=main attempt=1 stderr="! [remote rejected] main -> main (failed)"`.
The stderr is `remote rejected ... (failed)`, not a non-fast-forward rejection, which points at the remote refusing concurrent ref updates rather than a stale local ref.
