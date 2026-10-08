# Git network calls time out; Finalize warns only for config the task changed

`fabric-net-and-config-warn` · Task · bug, fabric

## Goal

Git network calls cannot hang a run, and Finalize warns about per-worktree config only when the task's copy differs from the parent's.

## From `git-network-timeout`

Imported from [issue #458](https://github.com/Knatte18/loomyard/issues/458).

### What happened

A `lyx batten run review-lens-fan` began a best-effort status push to the weft remote at 14:11:21; the laptop hibernated four seconds later.
After resume the push hung on a dead HTTPS connection (`git send-pack` in `futex_do_wait`) for 2.5 hours, and the batten run sat at Run-Shed without starting the child loom, with no notice to the parent.
Killing the `git push` let batten log `status push failed, next transition will catch up` and go on.

### Expected

Git network calls from lyx (push and fetch, best-effort or not) run under a timeout, or with `http.lowSpeedLimit`/`http.lowSpeedTime` set, so a dead connection fails within minutes and the best-effort path takes over.

## From `finalize-config-warn-from-main`

Imported from [issue #450](https://github.com/Knatte18/loomyard/issues/450).

At Finalize, loom warns: `task branch "<slug>" changed per-worktree config files since it forked from "main": _lyx/config/batcher.yaml (base <sha>, tip <sha>); landing does not carry them to the parent branch, so re-apply each change meant for the parent on the parent's copy`.

Seen for `shuttle-state` and `plan-glyph-precision` on 2026-10-08.
In both, the pair's `batcher.yaml` was byte-identical to the prime's: the change was main's own commit (`batcher: cautious is the template default`, c0bf19c54) arriving in the pair through the weft sync, not an edit the task made.

Expected: the check compares the tip against the parent's current copy (or excludes changes the weft sync brought in), and warns only when the task's copy differs from the parent's.
As it is, the warning fires on every run that forked before a config change on main, and the hub must diff each time to rule it out.
