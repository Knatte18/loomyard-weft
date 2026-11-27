## Goal

lym stops runs and mails the orch because a push to origin or a GitHub call fails on a flaky network, above all on VPN.
After this task lym and its agents only commit, lym's one background pusher brings everything lym owns to origin with a quiet, adaptive retry,
a landing is pushed at once but never fails on a transient error, and only a refusal that retrying cannot fix stops anything.

Observed faults, each reproducible on a network that resets HTTPS connections (`Recv failure: Connection reset by peer`, git exit 128):

- Issue #238: every failed detached push of `lym fabric sync`, `lym fabric add` or `lym board sync` mails the orch, eight messages an hour on one hub.
  Reproduce: make origin unreachable and commit on the board or a pair through those verbs.
- Issues #243 and #240: PR-Review and Finalize halt on an unreachable origin or a GitHub API `unexpected EOF` that outlasts about two minutes.
  Reproduce: keep origin or the GitHub API unreachable for more than two minutes while a run is at Finalize.
- The dobby's catch-up before a run start fails on `fabricengine: push lock busy ... neither side was pushed` when several tasks start at once.
  Reproduce: hold the push lock of a records worktree past 30 s while its run starts.
- Issue #242: Finalize blocks when its merge-in deleted a tracked directory that still holds files the task worktree ignored, such as `bin/` and `obj/`.
  Reproduce: merge in a parent commit that deletes a project whose ignored build output stands in the task worktree.
- A rejected push of a pair's code side can force origin past a commit made by hand on GitHub.
  Reproduce: add a commit on GitHub to a task's code branch, then push the pair.

## Scope

In:

- One pusher for every branch lym owns: both branches of every pair, beam pairs included, the board's branch and a landing's parent branch.
- Adaptive push timing in config, the owed ledger, the summary notices and the reed `sync:` field.
- Finalize's local landing, its owner check, the Go driver's network re-step, Worktree-Teardown and the dobby's catch-up under the rules below.
- Refusal classification, including the hand-commit guard on every push path that can reset or force origin.
- Finalize's merge-in leftover removal (issue #242).
- The agents' instruction that a failed push needs no action.
- The review findings `async-run-pusher` (`b4236816a`) left open.

Out:

- Branches the operator made outside lym, and a commit on the prime's code branch that no landing made.
- A fetch failure in the dobby's catch-up: that fetch is the owner verdict PATTERN-owner-machine requires, and this task keeps it.
- Moving push rules into `internal/fabriccore`, which `fabric-core-push-origin` owns after this task.

## Decisions

### D1 — One pusher pushes everything lym owns

`internal/runpusher` gains two target kinds beside the prime and a pair: the board's branch, and a landing's parent branch.
`lym board sync`'s detached push, and the detached pushes of `lym fabric sync`, `lym fabric add`, an anchor change and a lift, start or wake the pusher for their target instead of pushing themselves.
A pair target is owed whenever either branch is ahead of its origin tracking ref, whatever made the commits, plain `git commit` and a landing's merge on a beam branch included.
The prime's code branch is pushed only to a tip a landing recorded as owed, fast-forward only, never forced.
No lym path pushes a branch the operator made outside lym.
These pushes keep their own synchronous push, since a caller reads its result:
the board owner write, which PATTERN-owner-machine re-reads after its push, the board pushes of `internal/hubreconcile/walk.go` and `internal/tendercli/deps.go`, and webster reset's records push, whose owed record holds the run at its transition.

Rationale: the operator decided that lym pushes what it owns through the one pusher, so a failure surfaces in one place with one retry rule.

### D2 — The push interval is config and adapts to failures

Two keys in `internal/fabricengine/template.yaml` set the pusher's timing:
`fabric.push_interval_s`, 60 by default, and `fabric.push_ceiling_s`, 300 by default.
The ledger (D3) keeps each target's last push attempt and interval in force, so the timing holds across pusher processes.
A pusher pushes a target at most once per interval in force while the target is owed;
a pusher started within the interval waits out the rest of it.
A failed push doubles the interval in force for that target, up to the ceiling, and a push that lands returns it to `fabric.push_interval_s`.
A running pusher's wait also wakes, through `internal/fswatch`, on a push-now mark its target's ledger entry carries; the mark makes the next push skip the interval wait once.
A value below one second is floored to one second with one warning naming the key, as PATTERN-daemon-wakeups requires.
The keys are new, so no reinterpretation or move table ships.

Rationale: the operator asked for fewer pushes and less traffic, with a short interval while pushes go through and a longer one during an outage.

### D3 — An owed ledger feeds one summary notice and one all-clear

The ledger is one machine-local file per target under the hub's `.lym`, named through `internal/lymdirs`, written through `internal/state`, and one hub file for the time the hub last had no owed target.
A target's entry holds the time it was first seen owed, its last push attempt, its interval in force and its push-now mark.
The pusher writes the entry when it finds its target owed, and so does the dobby tick's ahead check when it starts a pusher.
The entry goes when the push lands, when the target's worktree is removed, handed over or cancelled away, and on a dobby tick that finds the target's worktree gone.
Adding the first entry and removing the last one happen under one ledger lock, named through `internal/lock`, which also guards the hub file.
A target whose push is refused leaves the ledger; its failed-push record and notice (D7) carry it.
The prime's dobby reads the ledger on its tick.
Once some target has stayed owed for `fabric.push_notice_after_min`, 30 by default, it posts the hub orch one notice listing every owed branch with the minutes each has been owed, and every run the Go driver is waiting on (D6) with its producer.
Once the ledger is empty and no run is waiting after such a notice, it posts one notice that every branch is pushed.
No more summary notices follow in the same owed period.
The pusher's own notice when its wait reaches the ceiling is removed.
An owed push still writes no failed-push record, so issue #238's per-push mail ends.

Rationale: the operator decided on no mail per failed push, one summary past a threshold and one notice when everything is pushed again.

### D4 — The reed bar shows `sync: <minutes>m · <seconds>s`

Every session bar of a hub shows one field, such as `sync: 3m · 60s`.
The minutes count from the time the hub last had no owed target, and read 0 while nothing is owed.
The seconds are the longest interval in force across owed targets, or `fabric.push_interval_s` while nothing is owed.
The field is a tmux `#()` job in `internal/reedengine/bar.go`, as the Slack toggles are, and the hub directory reaches it through a server-global option set when the hub's server starts.
The job runs a hidden verb, `lym fabric sync-status` (new), which reads the ledger, writes nothing, spawns nothing and touches no network.

Rationale: the operator decided the bar shows, before a machine switch, whether work is still only local.

### D5 — A local landing merges, then hands its push to the pusher at once

Before its local merge, Finalize fast-forwards a local parent that lags origin's, such as after a pull-request landing on the same parent; a local parent diverged from origin's is a refusal (D7).
It merges under `internal/parentlock`'s lock, records the parent tip as owed with a push-now mark on the landing target, starts or wakes that pusher and releases the lock.
Its records commit goes through the pair's pusher, and it pushes neither the parent nor its records synchronously, so no transient push failure fails it.
The landing target pushes the parent first and the board `done` only once the parent push landed, so origin never reads an ended entry whose parent lacks the run's work.
While the landing target owes a parent push, the run reader's past-the-merge read reads the local parent, so a resume after the merge never re-drives it.
A failed landing push falls back to D2's backoff.
A landing that needs GitHub, such as a pull request, checks before it takes the parent lock that its parent owes no landing push.
An owed one is a `git-transport` stop, which D6 waits out; a refused one is a blocked halt naming that refusal's way forward.
Other runs keep committing locally throughout.

Rationale: the operator decided that a landing pushes at once but a network outage never stops it or other work.

### D6 — The Go driver waits out a network stop instead of handing it back

A step stop with the transient class `git-transport` or `github-api` is re-stepped without a count cap.
The wait before each re-step starts at 15 s and doubles up to `fabric.push_ceiling_s`.
The wait runs between steps, so a waiting run holds no parent lock.
While it waits the driver holds its run lock and keeps a wait record naming the producer and the class, and it writes no halt note and hands nothing back.
`lym loom status`, the halt classifier in `internal/loomcli/halt.go` and the dobby's run watch read a run whose wait record stands under a held run lock as waiting, not as a stop, and mail nothing for it; the waiting run shows only in D3's summary.
A wait record whose run lock is free is a driver that died: the run watch reports it once as the stop it is.
Finalize's owner check wraps an unreachable origin as a `git-transport` stop instead of a blocked halt; `boardengine.ErrOwnerOriginUnreachable` itself is unchanged, so the catch-up and `lym adopt` keep their behaviour.
`agent-start` and `host-resources` keep today's path: the loop's one re-step, then the tackler.
The doc comment of `loomengine.AnomalyTransientStop`, the `loomcli` package doc and the reflection template say that only `git-transport` and `github-api` are re-stepped with waits (finding F2 of `transient-stop-not-failed`).

Bound: a network-class stop is re-stepped for as long as it stays network-class.
A refusal (D7) or any other class leaves this rule at once, D3's summary makes a long wait visible, and `lym batten cancel` stops it.

Rationale: the operator decided that a landing needing GitHub waits for the network and never fails on a transient error.

### D7 — A refusal stops at once, and no push resets past a commit lym did not make

A rejected push goes first through today's takeover read and rebase-free retry; what stays rejected after them is a refusal, as are an expired or refused login and a moved branch.
A refusal stops its pusher and its step at once and leaves one failed-push record and one notice, as `internal/fabricengine/failedpush.go` does today.
The classifier in `internal/shedtransient/shedtransient.go` and fabric's push classes read a git authentication failure (`Authentication failed`, `could not read Username`) and a GitHub 401 as refusals, never as owed or transient.
`resetUnnamedTip` in `internal/fabricengine/pushclassified.go` resets origin's code branch only when every commit origin holds past the named tip is reachable from a local branch or from the local code branch's reflog;
remote-tracking refs and their reflogs do not count, since every fetch fills them with origin's commits.
Otherwise the push is refused and its notice names the first such commit.
The same rule binds every other push path that resets or forces an origin branch, including each path the change description of `pr-modes-no-agent` names.
After a takeover, the new owner lacks the old owner's local refs, so a reset there is refused and named; that stop is accepted.

Rationale: the operator decided that a refusal stops at once with one notice;
a commit no local branch or reflog of this machine reaches was not made by lym here, so resetting past it drops someone's work.

### D8 — The dobby's catch-up hands an owed push to the pusher

When the catch-up's push of a run's owed commits fails as owed, a held push lock included, the catch-up starts that run's pusher and starts the run.
A refusal still fails the catch-up with its way forward.

Rationale: a held push lock is contention, not a failure, and the push is the pusher's to finish.

### D9 — Finalize's merge-in removes ignored leftovers of a deleted directory

After its merge-in, and again on every resume past it, Finalize finds each directory whose tracked files the merge commit deleted against its first parent.
It removes each untracked file under such a directory that the first parent's ignore rules, read from the first parent's tree, ignore, and logs each removal.
Any other dirty or untracked path still halts with today's reason.

Bound: only files the first parent's rules ignored, under a directory the merge emptied of tracked files, are removed; a tracked change or a file those rules do not ignore is never touched.

Rationale: such files are build output of a project that no longer exists, and the run has no agent left to remove them (issue #242).
Reading the rules from the merge commit's first parent makes the removal work on a resume after the merge.

### D10 — Agents are told a failed push needs no action

The edit directives of both engines, `contracts/stencils/engine/claude/edit-directive.md` and `contracts/stencils/engine/codex/edit-directive.md`, say that an agent may push, that a failed push needs no action, and that lym pushes its commits later.
No row or gate requires an agent's commits to be on origin: no production file under `internal/gateslot`, `internal/gatecli`, `internal/cardverify` or `internal/verifytree` reads origin, and lym's own pushes in PR-Review and Finalize follow D5 and D6.

Rationale: the operator decided that an agent's failed push never stops it.

### D11 — The review findings left by `async-run-pusher` are closed

Tests pin the pusher's lock and start conditions, the pusher restarts from the dobby tick and the batten and loom resumes, and the teardown's push and level wiring.
A test pins that Worktree-Teardown with git or pushes off pushes nothing and removes the pair once, with no loop.

Rationale: the entry asks to check each finding and close what is still open.

### D12 — The parts the pusher already settles are named and dropped

Issue #218: `internal/runfsm/act.go` no longer holds a run whose pair stands on this machine and holds the code tip, since its pusher owes the push.
Issue #254: the loomcli "status push is owed" loop is gone, since transitions no longer push, so no step stalls on a status push and no in-flight step and producer can disagree over one.
#254's ask to bound the owed retry is replaced by the operator's quiet retry: the pusher retries without bound, and D3's summary and D4's field make an outage visible.
The change description names both as settled by `async-run-pusher`, says #254's bound is replaced, and changes no code for them.

Rationale: the entry asks that a part the pusher already settles be named in the change description and dropped.

### D13 — Worktree-Teardown hands an unpushed pair to the dobby

Worktree-Teardown pushes nothing itself: it sets the pair target's push-now mark and starts or wakes its pusher.
When origin's tracking refs already hold both tips, it removes the pair as today.
Otherwise the row is done and leaves the pair standing.
The dobby's landed-pair teardown, `tearDownLeftover` in `internal/beamcli/dobbydeps.go`, removes a landed pair only once origin's tracking refs hold both tips and the landing target owes no push for its parent; until then it leaves the pair for a later tick.

Rationale: a teardown that waits for the network holds its run on one row, and one that removes an unpushed pair loses the work on a machine switch.

## Constraints

- Root cause, from the code at `3fbf277ca`:
  - Issue #238: the detached pushes in `internal/fabricengine/spawn.go` and `internal/boardengine/spawn.go` record each failure through `internal/fabricengine/failedpush.go`, whose notifier, `internal/orchcli/pushnotifier.go`, mails per failure episode and per branch; they do not go through `internal/runpusher`.
  - Issues #243 and #240: `internal/sheddriver/driverules.go` re-steps a network stop after 15 s, 30 s and 1 min, then hands back.
    `internal/landingshed/finalize_owner.go` halts on an unreachable origin.
    `internal/landingshed/finalize.go` pushes the parent and the records synchronously after the merge.
  - Push lock busy: `internal/battencli/catchup.go` pushes owed commits with a 30 s lock wait and fails the catch-up on any error.
  - Issue #242: the clean-tree check in `internal/landingshed/verifygate.go` counts every untracked path, and nothing removes ignored leftovers.
  - Hand commits: `resetUnnamedTip` force-resets with a lease at the tip it read after a fetch, which a hand commit satisfies.
- PATTERN-pair-remote: the records side goes first and the code side only once it landed, for every target D1 adds.
- PATTERN-landing: the local parent merge stays inside Finalize under `internal/parentlock`'s lock; only the push leaves it.
- PATTERN-owner-machine: the pusher's run-scope origin reconcile before each push stays.
- PATTERN-daemon-wakeups: the pusher's wait and the driver's network wait are listed unbounded waits; a test injects their clock and never waits out an interval.
- PATTERN-config-key-names: every message names the new keys in full through `configengine`'s key-name helper.
- PATTERN-tolerant-records: the ledger refuses no unknown key.
- PATTERN-lym-files: the ledger's files and lock are named through `internal/lymdirs` and `internal/lock`.

## Auto-mode assumptions

- The ledger, the bar field and the summary notice cover one hub, not every hub on the machine, since each hub has its own origin and orch.
- The prime's dobby runs whenever the hub has work, so it can carry the summary notices.
- The notice threshold is one key, `fabric.push_notice_after_min`, which also governs when a waiting run enters the summary.
- The driver's network wait starts at 15 s, today's first wait, and shares the pusher's ceiling.

## Open risks

- The entry asks for the minutes since "this machine" last had nothing owed; D3 and D4 cover one hub. The parent decides whether the field and the all-clear must cover every hub on the machine.
- A dobby catch-up whose fetch fails on an unreachable origin still stops that run's start (Scope, Out), although the entry says a network outage never stops other work; the parent decides whether that stays.
- A landing whose parent push stays owed holds every later pull-request landing on that parent (D5); D3's summary is its only signal.

## Acceptance criteria

- D1: a test pins that a commit through `lym board sync`, `lym fabric sync` and `lym fabric add` starts or wakes the pusher for its target and spawns no detached push of its own; tier1 or the lowest tier that fits.
- D1: a test pins that a pair target is owed for a code commit made with plain `git commit` past the last transition's named tip, and that the prime's code branch is pushed only to a tip a landing recorded, fast-forward only.
- D2: a test with an injected clock pins the interval in force: 60 s after a landed push, doubled after each failure to 300 s, back to 60 s after a push lands, kept across two pusher processes, skipped once by a push-now mark, and a sub-second value floored with one warning.
- D3: a test with an injected clock pins that the dobby posts one summary listing every owed branch and every waiting run once a target has stayed owed for the threshold, no second summary in the same owed period, and one all-clear once the ledger empties.
- D3: a test pins that an owed push writes no failed-push record and posts no notice of its own, that a refused target leaves the ledger, and that a removed pair's entry goes.
- D4: a test pins `lym fabric sync-status`'s output, `sync: <m>m · <s>s`, for an empty ledger and for owed targets with different intervals.
- D4, Review: `internal/reedengine/bar.go` renders the field in every hub session bar.
- D5: a test pins that a local landing merges under the parent lock, records the owed parent tip, starts the pusher and returns done while origin is unreachable, and that a resume after it reads the run as past its merge.
- D5: a test pins that the landing target pushes the board `done` only after the parent push landed.
- D5: a test pins that a pull-request landing whose parent owes a landing push stops as `git-transport` before taking the parent lock.
- D6: red-first test in `internal/sheddriver`: a `git-transport` stop whose repair count is past three is still re-stepped with a wait capped at `fabric.push_ceiling_s`, where the merge base hands it back; tier1.
- D6: a test pins that the driver writes no halt note and the dobby's run watch mails nothing while a wait record stands under a held run lock, and that the run watch reports one stop once the run lock is free.
- D6: a test pins that Finalize's owner check on an unreachable origin is a `git-transport` stop, not a blocked halt.
- D6, Review: the doc comment of `loomengine.AnomalyTransientStop`, the `loomcli` package doc and the reflection template name only `git-transport` and `github-api` as re-stepped with waits.
- D7: red-first test in `internal/fabricengine`: a rejected code push whose origin holds a fetched commit no local branch or reflog reaches is refused, naming that commit, where the merge base resets past it.
- D7: a test pins that an authentication failure is a refusal with one record and one notice.
- D8: red-first test in `internal/battencli`: a catch-up whose push lock stays held starts the pusher and the run, where the merge base fails the catch-up.
- D9: red-first test in `internal/landingshed`: a merge-in that deletes a tracked directory holding ignored files removes them and goes on, also on a resume after the merge, where the merge base halts; a tracked change still halts.
- D10, Review: both engines' edit directives say a failed push needs no action because lym pushes the commits later.
- D11: the tests D11 names exist, each pinning behavior no other test covers.
- D12, Review: the change description names issues #218 and #254 as settled by `async-run-pusher` and says #254's bound is replaced by the quiet retry.
- D13: a test pins that Worktree-Teardown with an unreachable origin ends done with the pair standing, and that the dobby's landed-pair teardown removes the pair only once origin holds both tips and the parent owes no landing push.

## PATTERN changes

- `PATTERN-pair-remote` — changes: Pushing anything lym owns or tearing down a pair: one background pusher per target, records side before code, which no transition waits for and which retries an owed push quietly; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.

## Notes for the plan writer

- The backgrounds of PATTERN-pair-remote, PATTERN-run-fsm, PATTERN-landing and PATTERN-daemon-wakeups describe the pusher's targets, the landing's synchronous pushes and the pusher's one-minute ceiling; each needs its background edit.
- The driver's network rule sits in `internal/sheddriver/driverules.go`, and its wait is applied in `internal/sheddriver/drive.go`.
- The Slack toggle in `internal/reedengine/slacktoggle.go` is the model for the bar's `#()` job.
- The run reader's past-the-merge read is in `internal/pairteardown/runread.go`.
