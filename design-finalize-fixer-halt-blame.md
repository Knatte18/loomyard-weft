# Finalize blames the fixer session for a push of origin/<task branch> it did not make

`finalize-fixer-halt-blame` · Task · bug, landing · running · Preflight

Imported from [issue #271](https://github.com/Knatte18/loomyard/issues/271).

## What happened

A `Finalize` merge fixer run ended `pushed` and halted the run for a human, although the fixer session ran no push.

The halt reason, with local paths removed:

```
landing stop: the fixer session of Finalize pushed origin/pr-modes-no-agent at b6eb9ac69a8d5e12c863dd0a5a27587e1ff0446a; session log: <run dir>; way forward: check the pushed commits are the task's, then `lym loom resume`
```

Trace excerpts, in order:

```
2026-11-26T22:39:50 fabric: mutation kind=merge_committed target=pr-modes-no-agent detail=b6eb9ac69a8d5e12c863dd0a5a27587e1ff0446a
2026-11-26T22:39:51 verifyrun: spawning verify command command="lym gate test --wait --gate finalize"
2026-11-27T07:21:40 verifyrun: verify command exited exitCode=1
2026-11-27T07:21:41 shuttle: run started role=fixer
2026-11-27T07:33:53 landingshed: fixer run ended row=Finalize mode=finalize end=pushed spent=1 cap=2
```

Reflog of the remote-tracking ref:

```
b6eb9ac69 refs/remotes/origin/pr-modes-no-agent@{2026-11-27 07:22:03 +0100}: update by push
1966e5525 refs/remotes/origin/pr-modes-no-agent@{2026-11-26 22:27:45 +0100}: update by push
```

- The pushed tip is the merge commit lym itself made before the fixer started.
- The fixer session's transcript holds no `git push` among its commands, which were reads, `lym config`, `lym gate test --gate finalize` and `git status`/`git log`. Its stencil forbids pushing.
- The push at 07:22:03 came from some other process. I found no lym trace that records it, so the pusher is not identified. Another lym process in the hub logged a DNS failure reaching github.com at 07:21:41, so connectivity had been down shortly before; a delayed retry is one hypothesis, not verified.

## Why it is abnormal

The end `pushed` is decided from `origin/<task branch>` differing before and after the session, and the halt text states that the fixer session pushed.
Any other writer of that ref during the session is blamed on the fixer.
Here the run stopped on a fix whose landing gate had passed, and escalated to a human for a push of lym's own commit.

## What to do

- Do not attribute a ref movement to the fixer when the new tip is a commit that existed before the session started, such as the task HEAD read at session start. Report only commits the session could have added.
- Word the halt as "the remote ref moved during the fixer session" unless the pusher is known.
- Make every push of a task code branch leave a trace line, so the writer can be found.

Provenance: task `pr-modes-no-agent`, note `loom-halt.md`.


## From issue #272

Imported from [issue #272](https://github.com/Knatte18/loomyard/issues/272).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `.lym/shuttle/`, `/.lym/shuttle/55336935c2b090f90010928604d00408`.

## What happened

A `Finalize` halt reason pointed the reader at the fixer session's log, as a run directory under `.lym/shuttle/<run id>`:

```
landing stop: the fixer session of Finalize pushed origin/pr-modes-no-agent at b6eb9ac69a8d5e12c863dd0a5a27587e1ff0446a; session log: <worktree>/.lym/shuttle/55336935c2b090f90010928604d00408; way forward: check the pushed commits are the task's, then `lym loom resume`
```

The same step removed that directory just before writing the reason:

```
2026-11-27T07:33:53.776 shuttle: run finished runDir=<worktree>/.lym/shuttle/55336935c2b090f90010928604d00408 outcome=done cleanedUp=true
2026-11-27T07:33:53.778 landingshed: producer stuck producer=Finalize reason="landing stop: ... session log: <worktree>/.lym/shuttle/55336935c2b090f90010928604d00408; ..."
```

Listing the directory afterwards answers "No such file or directory".

## Why it is abnormal

The halt asks a human or orch to judge what the session did, and names evidence that no longer exists.
The other fixer halts that print a session log (`timeout`, `no-report`, `dirty`) are built from the same value, so they are likely affected when the seat run finishes `done`; only the `pushed` case was observed.

## What to do

Either keep the run directory when the fixer run does not end `fixed`, or name something that survives cleanup in the halt reason, such as the engine's session id and transcript.

Provenance: task `pr-modes-no-agent`, note `loom-halt.md`.
