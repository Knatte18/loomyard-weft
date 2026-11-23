# Card 6 — loom-pusher-verb

**Create:**
- `internal/loomcli#loomCLI.pusherCmd`
- `internal/loomcli#pusherDeps`
- `internal/loomcli#TestPusherDeps`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#loomCLI.resolvePersistentPreRun`
- `internal/loomcli/doc.go`
- `internal/loomcli#TestResolvePersistentPreRun_SkipsSlugGroups`

**Uses:**
- `internal/runpusher#Run`
- `internal/runpusher#NewDeps`
- `internal/runpusher#TargetOf`
- `internal/runpusher#WorktreeFlag`
- `internal/runpusher#SlugFlag`
- `internal/loomcli#newPrePushCheck`
- `internal/loomcli#prePushCheck.guard`
- `internal/loomcli#primeOfHub`
- `internal/loomcli#stoppingPrePushCheck`
- `internal/pairteardown#PushRun`
- `internal/orchcli#NotifyParent`

**Intent:**
Add the hidden verb the pusher runs as, `lym loom pusher --worktree <code worktree> [--slug <slug>]`.
It lives in loomcli because a pair's pusher runs loom's pre-push check (D7), which is loomcli's `prePushCheck`.

- `pusherCmd` is hidden and named for the role audience.
  It resolves the worktree its `--worktree` flag names through `lymcwd.ResolveWorktree`, never the cwd,
  since the pusher starts in the hub's Bolt directory.
  It builds the target with `runpusher.TargetOf` and the prime with `primeOfHub`, then runs `runpusher.Run` with `pusherDeps`.
  A worktree that does not resolve fails the verb with the resolve error, its message ending in
  `; way forward: run lym loom pusher --worktree with the path of an existing code worktree; the next commit, dobby tick or resume starts the pusher again`.
- `resolvePersistentPreRun` skips `pusher`, as it skips `checklist`, so the verb arms no run.
- `pusherDeps` starts from `runpusher.NewDeps(t)` and fills:
  `Push`, through `pushRun` (`pairteardown.PushRun` in production, discarding its result)
  with `OwedUnrecorded` true and `fabricengine.StatusPushLockWait` for the push lock:
  a pair pushes its pair with its slug, `TeardownWait` of `StatusPushLockWait` and `InsidePair` true,
  so a takeover hands the run over leaving the pair for the parent's dobby (D6);
  the prime pushes its own records with no slug (cards 3 and 4);
  `Guard`, for a pair only, the guard of `check`, `newPrePushCheck(pair, shedrun.SelfRunID)` in production:
  the run-scope origin reconcile with today's verdicts, its Finalize skip and its stops (D7);
  `Notify`, a line through `orchcli.NotifyParent` from the target's location under `loomProgram`.
  The pusher reads the pause before the guard on every attempt (card 5), so the guard's own pause skip never lets a push through in the pusher.

Bound: the verb pushes only the worktree its flag names, through the same `PushRun` a transition used to call,
so it lets nothing reach origin a transition's push did not;
an agent running it by hand starts one more pusher, which the pusher lock turns away.
Bound: a guard stop for an entry another machine owns pauses nothing, as today's pre-push stop does,
so each later commit of the run starts a pusher that fetches and posts the stop again;
that repeats only until the dobby's catch-up hands the run over, as the record's D7 has it, and is accepted.

`loomcli/doc.go` names the verb and says a pair's pusher, not the commit, runs the pre-push reconcile.

`TestPusherDeps` is tier0 over hand-built locations, `stoppingPrePushCheck` and a recording `pushRun`:
a pair target's `Push` calls `pushRun` with the pair, its slug and a `RunPush` of `OwedUnrecorded` true, `InsidePair` true
and both waits at `StatusPushLockWait`, and its `Guard` stops with an error wrapping `errPushStopped`;
a prime target's `Push` calls `pushRun` with the prime, an empty slug and `InsidePair` false, and it carries no `Guard`;
both carry `Notify`.
`TestResolvePersistentPreRun_SkipsSlugGroups` is tier0 and gains a row:
`pusher --worktree <a missing directory>` exits 1 with the resolve error and its way-forward clause, not the pre-run's arming refusal.

**ImpactSummary:** One new hidden loom verb and one pre-run skip; no existing verb, check or caller changes.
