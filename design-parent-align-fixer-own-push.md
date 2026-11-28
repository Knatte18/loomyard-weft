# Parent-Align halts when its own fixer pushes the task branch, reading the fixer commits as a foreign move

`parent-align-fixer-own-push` · Task · bug, landing · draft

Depends on: [`landing-ledger-own-tip`](design-landing-ledger-own-tip.md)

## Seen

`kde-window-placement-build` halted twice at Parent-Align on 2026-11-28 (14:18 and 14:47 UTC) with
`landing stop: the remote ref origin/<slug> moved during the fixer session of Parent-Align to <sha>, which holds commits the task head <sha> the session started from lacks`.
Both times the new commits were the Parent-Align fixer's own fixes on the task branch (a guard rename, a hermetic-guard exemption, a doc comment), pushed by its session.
The orch checked the commits and resumed with `lym loom resume`; each halt cost a halt notice, an orch turn and the wait.

## Idea

Parent-Align tells its own fixer's commits from a foreign move: commits the fixer session made on the task branch, descending from the session's start head, pass without a halt.
Only a move that holds commits the session did not make stops the run.


## From issue #297

Imported from [issue #297](https://github.com/Knatte18/loomyard/issues/297).

## What happened

A `Finalize` merge fixer committed two fixes on the task branch and the landing gate passed over them.
The run then halted for the orch, because lym's own pusher had pushed those two commits to the task branch's remote ref while the fixer session was still running.

The halt reason:

```
landing stop: the remote ref origin/<task branch> moved during the fixer session of Finalize to 9663d3073f3871d95b752bf994a232e691862213, which holds commits the task head 8169ba2b6d932c03be922f42986da6419d95521e the session started from lacks; session: claude session 48927808-ca1f-49c4-b067-366f4f2b2afa; way forward: check the commits 8169ba2b6d932c03be922f42986da6419d95521e..9663d3073f3871d95b752bf994a232e691862213 are the task's, then `lym loom resume`
```

Timeline, local time 2026-11-28, from the run's traces and the worktree's reflog:

```
12:00:55  fabric: mutation kind=merge_committed detail=8169ba2b...     (task head the fixer starts from)
12:13:52  shuttle: run started role=fixer sessionID=48927808-...
12:20:51  two commits by the fixer session; HEAD = 9663d3073
12:20:58  argv=loom pusher --worktree <task worktree> --slug <slug>    (records side: push landed)
12:21:02  gitrepo: push landed ref=refs/heads/<task branch> tip=9663d3073...   (same pusher process)
12:21:09  shuttle: session state idle-done
12:31:35  landingshed: post-merge landing gate passed
12:31:37  landingshed: fixer run ended row=Finalize mode=finalize end=pushed spent=1 cap=2
12:31:37  landingshed: producer stuck
```

The fixer session's transcript holds commits and no `git push`.
The pusher process carries the same trace id as the hub's `reed watchdog` daemon.

## Why it is abnormal

The moved ref held only the commits this session made, pushed by lym itself, which is ordinary behaviour for a commit on a task branch.
`internal/landingshed/fixerrun.go` excuses a moved ref only when the new tip is an ancestor of the head the session started from.
Its comment gives the reason as "The session cannot have added such a commit".
Any commit the fixer does add, once the pusher sends it before the session ends, therefore reads as `pushed`.
So a fixer that succeeds can still stop the run and need an orch to resume it.

This follows #271, whose case was a push of lym's own merge commit made before the session started.
The exclusion covers that case and not this one.

The fixer's prompt also contradicts itself: `landing-template-fixer.md` says "Never push", while the edit directive rendered into the same prompt says "You may push your own worktree's commits".

## What to do

- Do not end `pushed` when the new remote tip is the task head, or an ancestor of it, as read after the session: those commits are in the task worktree and are what the landing gate judged.
  Alternatively, hold lym's pusher off the task branch while a landing fixer session runs.
- Keep the stop for a remote tip that holds commits the task worktree does not have, and for a deleted ref.
- Settle which of the two push rules the fixer follows.

Provenance: task `gatecli-dotnet-redfirst-time`, note `loom-halt.md`.
