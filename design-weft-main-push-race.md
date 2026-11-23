# Find who pushes weft main outside the board push lock

`weft-main-push-race` · Task · research, fabric, board, batten

## Goal

Find why pushes of the weft repository's `main`, the board, race each other on one machine, and name the fix.

## Seen

On 2026-11-23 around 09:20, with several runs starting at once on Neve:

- The Bolt checkout's board push failed: `git push https://github.com/Knatte18/loomyard-weft.git refs/heads/main:refs/heads/main` was refused with `cannot lock ref 'refs/heads/main': is at 9f73b6e9 but expected f4a24e2b`, and `gitrepo` gave up after 4 attempts.
- A `lym batten run` catch-up minutes before logged the same refusal on `ref=main`, beside a records push to `loomyard-weft`.
- A start was refused with `push lock busy: .../loomyard-weft/.weft/fabric.push.lock still held after 30s`.
- `lym board sync` then pushed cleanly.

By design, board writes take the board write lock and every push of the board goes through `coalescePush` under the absorbing `board.push.lock` (`internal/fabricengine/coalesce.go`, `bolt.go`), so one process pushes the latest commit for all waiting writers.
A refusal whose ref moved between read and push means a second pusher of weft `main` acts outside that lock, or under another lock.

## Questions

- Which code paths push weft `main`: list every call site, the lock each takes (`board.push.lock`, `fabric.push.lock` or none), and from which checkout (the Bolt checkout, `loomyard-weft`, a task's records worktree).
- Which of them ran at 09:15 to 09:25 on 2026-11-23, from the hub log and the runs' traces.
- Whether another machine wrote the board in that window.
- Why the retry gives up: does each retry re-read origin and rebase or merge, or push the same stale commit again?

## Bounds

- Read code, logs, traces and git history; no timed runs and no trials.
- The report ends with the cause and a concrete fix for each path, so every push of weft `main` on a machine is coalesced under one lock, as a candidate note.
