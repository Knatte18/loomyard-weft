## Goal

lym stops runs and mails the orch because a push to origin or a GitHub call fails on a flaky network, above all on VPN.
After this task lym and its agents only commit, lym's one background pusher brings everything lym owns to origin with a quiet, adaptive retry,
a landing waits while origin cannot answer its owner checks and, once origin answers, merges locally and hands its push to the pusher,
so no transient push failure fails it, and only a refusal that retrying cannot fix stops anything.

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
  Reproduce: add a commit on GitHub to a task's code branch, commit on the local code branch, then push the pair.

## Scope

In:

- One pusher for every branch lym owns: both branches of every pair, beam pairs included, the board's branch and a landing's parent branch.
- Adaptive push timing in config, the owed ledger, the summary notices and the reed `sync:` field.
- Finalize's local landing, its owner check, batten's Land row and every other pull of the prime, the Go driver's network re-step, Worktree-Teardown and the dobby's catch-up under the rules below.
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
A landing target exists only for a parent branch no pair target serves, the prime's code branch;
a landing onto a beam parent records its landing entry (D5) and push-now mark on the beam pair's target,
whose pusher pushes the beam's records side, then its code branch, under the beam pair's one pusher lock.
Every detached push spawn, `fabricengine.SpawnDetachedPush`, `fabricengine.SpawnStatusPush` and `boardengine.SpawnSync`,
starts or wakes the pusher for its target instead of pushing itself, whichever verb or package calls it.
A pair target is owed whenever either branch is ahead of its origin tracking ref, whatever made the commits, plain `git commit` and a landing's merge on a beam branch included.
The prime's code branch is pushed only to a tip a landing recorded as owed, fast-forward only, never forced.
No lym path pushes a branch the operator made outside lym.
Every synchronous push keeps its push, each for the reason its caller reads the result:

- the board owner write, which PATTERN-owner-machine re-reads after its push;
- `lym config`'s hub-wide commit through `bolt.Push` in `internal/configcli/hubwide.go`, which reports whether the hub-wide config reached origin;
- webster reset's records push, whose owed record holds the run at its transition;
- the dobby's catch-up push (D8), which reads whether the push landed, stayed owed or was refused before the run starts;
- PR-Review's task-branch push through `pairteardown.PushRun`, since GitHub needs the branch on origin before it opens or merges the pull request.

A synchronous push of a run's pair records sends neither a records commit a landing entry holds (D5) nor any commit after it.

The board target's rules are unchanged: a landing's `done` board write is made only once its parent push landed (D5), so no board push needs a hold.

Rationale: the operator decided that lym pushes what it owns through the one pusher, so a failure surfaces in one place with one retry rule.

### D2 — The push interval is config and adapts to failures

Three keys in `internal/fabricengine/template.yaml` set the pusher's timing and notices:
`fabric.push_interval_s`, 60 by default, `fabric.push_ceiling_s`, 300 by default, and `fabric.push_notice_after_min`, 30 by default, which D3 reads.
The ledger (D3) keeps each target's last push attempt and interval in force, so the timing holds across pusher processes.
A pusher pushes a target at most once per interval in force while the target is owed;
a pusher started within the interval waits out the rest of it.
The pusher's owed retry is the interval in force alone:
`runpusher.PushUntilLanded`'s one-second-to-a-minute loop and its ceiling notice are replaced by one push per interval,
and the pusher's pushes run `pushClassified` with no transport backoff of their own.
A failed push doubles the interval in force for that target, up to the ceiling, and a push that lands returns it to `fabric.push_interval_s`.
A running pusher's wait also wakes, through `internal/fswatch`, on a push-now mark its target's ledger entry carries; the mark makes the next push skip the interval wait once.
A value below one second is floored to one second with one warning naming the key, as PATTERN-daemon-wakeups requires.
The keys are new, so no reinterpretation or move table ships.

Rationale: the operator asked for fewer pushes and less traffic, with a short interval while pushes go through and a longer one during an outage.

### D3 — An owed ledger feeds one summary notice and one all-clear

The ledger is one machine-local file per target under the hub's `.lym`, named through `internal/lymdirs`, written through `internal/state`, and one hub file for the time the hub last had no owed target.
A target's entry holds the time it was first seen owed, its last push attempt, its interval in force and its push-now mark.
The pusher writes the entry when it finds its target owed, and so does the dobby tick's ahead check when it starts a pusher.
The entry goes when the push lands, when the target's worktree is removed, handed over or cancelled away,
on a dobby tick that finds the target's worktree gone, and on a dobby tick whose ahead check finds the target level, whoever pushed it.
A parent target's entry stays while any of its landing entries (D5) stands, so the all-clear waits until every landing is settled or dropped.
Adding the first entry and removing the last one happen under one ledger lock, named through `internal/lock`, which also guards the hub file.
A target whose push is refused keeps its entry while its refused-push record (D7) stands.
The ledger stores no refused mark: a target reads as refused while a refused-push record of one of its own branches stands, read per target.
The landing target reads the records of the prime's code branch, which `fabricengine.RefusedPushStands` skips for a prime location today,
and a refusal of the prime's records branch counts against the prime's target alone.
Clearing that record through the way forward it names ends the refusal,
such as a `lym fabric push` that lands for a pair's branch, a beam parent's included, or a `git push` in the prime's worktree that lands for the prime's code branch (D5);
the next dobby tick then finds the target level and drops its entry, or finds it still ahead and starts its pusher.
The dobby's ahead check writes no new entry for a target whose refused-push record stands.
The prime's dobby reads the ledger on its tick.
Once some target has stayed owed, or some run has kept a wait record (D6), for `fabric.push_notice_after_min` (D2),
it posts the hub orch one notice listing every owed branch with the minutes each has been owed, refused ones named as refused,
and every run the Go driver is waiting on with its producer.
Once the ledger is empty and no run is waiting after such a notice, it posts one notice that every branch is pushed.
The owed period counts from the first owed entry or wait record, whichever came first, and no more summary notices follow in it.
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

The parent target is the target that pushes a landing's parent branch: the landing target for the prime's code branch, the beam pair's target for a beam parent (D1).
A landing entry is the parent target's record of one landed run:
it names the run, its landed parent tip, the records commit its pair target holds and its pending `done` write.
A later landing on the same parent adds its own entry, and the parent target pushes the newest owed tip.
The parent target owes a parent push while any of its landing entries stands, refused or not.

A local landing is judged against origin before it merges, as PATTERN-owner-machine requires.
Its owner checks' run-scope reconcile in `gateUnlocked` and `landUnderLock` and `handedOverStop`'s owner reads stop as `git-transport` while origin is unreachable (D6),
and Finalize marks a transport failure of `syncParent`'s fetched tips read as `git-transport` too, where today it is a plain error;
D6 waits them out, and the landing merges once origin answers them.
Before its local merge, Finalize fast-forwards a local parent that lags origin's, such as after a pull-request landing on the same parent;
a local parent diverged from origin's is a refusal (D7).
A local parent ahead of origin by only tips its parent target records as owed is landed on as an equal parent, and the landing adds its own landing entry;
any other ahead commit keeps today's Stuck reason.
It merges under `internal/parentlock`'s lock, records its landing entry with a push-now mark on the parent target, starts or wakes that pusher and releases the lock.
Every Finalize pass that finds the local parent holding the task's work while origin's parent lacks it records that landing entry, the resume included,
so a Finalize killed between the merge and the record still leaves the push owed.
With git or pushes off no landing entry is written, and Finalize makes its `done` board write itself, as today.
Its records commit goes through the pair's pusher, and it pushes neither the parent nor its records synchronously, so no transient push failure fails it.

Finalize makes no `done` board write while its landing entry stands.
Once origin's parent holds an entry's tip, the parent target's pusher makes that entry's `done` write,
which wakes the board target like any board write, then releases its pair hold, sets the pair target's push-now mark and removes the entry;
it settles entries one by one in landing order, so a landed newer tip settles every older entry with it.
The landing entry holds the run's records commit Finalize makes, and the hold binds every push of the run's pair records, not only the pair's pusher:
the pair pusher, the catch-up's `pairteardown.PushRun` (D8), PR-Review's record commit push and cancel's records push send neither that commit nor any after it while the entry stands.
The catch-up treats the commits the hold keeps back as D8 says.
So origin never reads the run's pair records or an ended entry whose parent lacks the run's work, whichever push carries the board branch,
and no teardown of the run's pair runs while the entry stands (D13).
Batten's run record lives in the prime's records, which every run shares, so it is not held and reaches origin at Land and Worktree-Teardown:
a reader that judges the batten record at Land or Worktree-Teardown, the prime's local one or origin's, the run reader's past-the-merge read of the local one among them,
reads the run as past its merge only when the parent it judges holds the run's work, the parent chosen by the paragraph below on the past-the-merge read.
A landing entry whose owed tip origin's parent already holds, however it got there, is settled on the next pusher pass or dobby tick:
the `done` write is made, the pair target's hold is released, the parent's refused-push record is cleared, and the entry goes.
So a refused parent push whose way forward is a push by hand clears once that push lands.

The parent target's pusher reads owners before each parent push and before each `done` write.
Before a parent push it reads the parent's owner and the owner of every run whose landing entry the push carries, every entry up to the tip it pushes;
before a `done` write it reads that entry's run's owner again.
The landing target first pulls the Bolt from origin through `boardengine.PullBolt` and reads every owner from it, so it never acts on a verdict read before an absence;
a beam parent's push reads the beam's owner and each run's owner through the beam pair's run-scope reconcile.
It pushes and writes only while no other machine owns the parent or any run the push carries.
An owner read that cannot reach origin is an owed push, retried at D2's interval.
A parent or a run owned elsewhere is a refusal (D7): the pusher writes one refused-push record for the parent's branch, naming the owning machine and, for a run, the run,
which keeps later ticks from starting the pusher and holds the notice to one, and it leaves the landing entries standing.
A run owned elsewhere holds back every recorded tip that carries its merge;
an older entry's recorded tip, which does not, is still pushed and settled.

The refusal's way forward names the parent target's push:
`git push` in the prime's worktree for the prime's code branch, whose pair push is records-only, and `lym fabric push` for a beam parent.
For a parent owned elsewhere it is `lym adopt` of the parent on this machine, `root` for the prime, then that push;
for a run owned elsewhere it is `lym adopt` of the run's subject, its beam or its slug, on this machine, then that push,
or leaving the run to its owner: once the owner lands it, the next pass drops the landing as below.
Once the push lands, the settle rule above makes the `done` writes and clears the record.
When origin's parent moved on while the other machine owned it, that push is refused as diverged (D7), naming the landing commits origin lacks;
its way forward is to merge origin's parent into the local parent in the parent's worktree, then the parent target's push.

Before each parent push the pusher reads, for each standing entry, whether origin's parent already holds the run's work under another commit, the read `adopterFirstRepair` makes today,
as after a new owner re-ran Finalize from origin; such a landing is dropped by the pusher instead of pushed, since a merge or a push would land the work twice.
The drop removes the entry with no `done` write, since the landing origin holds made its own, and clears the parent's refused-push record;
the entry's pair hold outlives it, so no push sends the held records commit or any after it to a branch, and the hold goes with the pair.
A run owned elsewhere is then handed over by the pusher through `pairteardown.HandOverOwnedElsewhere`, which removes the pair:
its pushes of the run's records stop short of the held commit, which reaches origin only under the hand-over's owner archive tag,
and it resets the local parent to origin's when the run's landing commit alone puts it ahead, as today.
A run this machine owns keeps its pair until `lym fabric remove`, which D13 bounds;
the pusher resets the local parent to origin's, `git reset --keep origin/<parent>`, when the landing commit alone puts it ahead.
When other local commits stand on the landing commit, no reset is made: the parent push is refused (D7), naming the commit and a rebase in the parent's worktree that drops it,
and the next pass after that rebase reads each standing entry's owed tip again as the local parent commit that merges its run and pushes the newest one.
The same refusal stands for any parent push whose tip carries a landing commit no entry records.

Every pull of the prime, batten's Land row and `lym adopt`'s deploy among them, follows the same rule while the parent target owes a parent push:
it reads a local prime code branch ahead of origin by only tips the parent target records as owed as level and moves no code side over them.
Batten's Land row, which pulls the prime right after Finalize, reads an unreachable origin at that pull as a `git-transport` stop, which D6 waits out,
instead of a Stuck, whether or not a push is owed.

While the parent target owes a parent push, the run reader's past-the-merge read reads the local parent,
and whenever origin's parent lacks the work while the local parent holds it, so a resume after the merge never re-drives it.
A failed landing push falls back to D2's backoff.
A landing that needs GitHub, such as a pull request, checks before it takes the parent lock that its parent owes no landing push.
An owed one is a `git-transport` stop, which D6 waits out;
a refused one is a blocked halt naming that refusal's way forward.
Other runs keep committing locally throughout.

Rationale: the operator decided that a landing pushes at once but a network outage never stops it or other work;
PATTERN-owner-machine keeps the owner verdict on origin, so only the verdict waits for the network.

### D6 — The Go driver waits out a network stop instead of handing it back

A step stop with the transient class `git-transport` or `github-api` is re-stepped without a count cap.
The wait before each re-step starts at 15 s and doubles up to `fabric.push_ceiling_s`.
The wait runs between steps, so a waiting run holds no parent lock and no run lock.
While it waits the driver holds its driver lock, `shedrun.DriverLock`, and keeps a wait record naming the producer and the class, and it hands nothing back.
A step the Go driver runs writes no halt note for a failed stop whose transient class is `git-transport` or `github-api`;
the driver writes one only when it stops waiting on a stop that is no longer network-class.
A step run outside the Go driver keeps today's halt note, since no driver waits on its stop.
`lym loom status`, the halt classifier in `internal/loomcli/halt.go` and the dobby's run watch read a run whose wait record stands under a held driver lock as waiting, not as a stop,
and mail nothing for it; the waiting run shows only in D3's summary.
A wait record whose driver lock is free is a driver that died: the run watch reports it once as the stop it is.
Finalize's owner check wraps an unreachable origin as a `git-transport` stop instead of a blocked halt; `boardengine.ErrOwnerOriginUnreachable` itself is unchanged, so the catch-up and `lym adopt` keep their behaviour.
`agent-start` and `host-resources` keep today's path: the loop's one re-step, then the tackler.
The doc comment of `loomengine.AnomalyTransientStop`, the `loomcli` package doc and the reflection template say that only `git-transport` and `github-api` are re-stepped with waits (finding F2 of `transient-stop-not-failed`).

Bound: a network-class stop is re-stepped for as long as it stays network-class.
A misconfigured origin host that never resolves, or a proxy that refuses every connection, is re-stepped like an outage
and surfaces only in D3's summary and through `lym batten cancel`.
A refusal (D7) or any other class leaves this rule at once, a GitHub rate limit and a 401, 403 or 404 response among them,
D3's summary makes a long wait visible, and `lym batten cancel` stops it.

Rationale: the operator decided that a landing needing GitHub waits for the network and never fails on a transient error.

### D7 — A refusal stops at once, and no push resets past a commit lym did not make

A rejected push goes first through today's takeover read and rebase-free retry; what stays rejected after them is a refusal, as are an expired or refused login and a moved branch.
A parent push D5 holds back is a refusal too: one whose parent or carried run another machine owns, and one whose tip carries a landing commit no landing entry records.
A refusal stops its pusher and its step at once and leaves one failed-push record and one notice, as `internal/fabricengine/failedpush.go` does today.
The classifier already reads a git authentication failure (`Authentication failed`, `could not read Username`) and a GitHub 401 as refusals, never as owed or transient,
and this task keeps that.
`resetUnnamedTip` in `internal/fabricengine/pushclassified.go` resets origin's code branch only when every commit origin holds past the named tip is reachable from a local branch or from the local code branch's reflog;
remote-tracking refs and their reflogs do not count, since every fetch fills them with origin's commits.
Otherwise the push is refused, and its notice names the first such commit and the way forward:
merge origin's branch into the task branch through the task worktree, then run `lym fabric push`.
The rule lives in `resetUnnamedTip`, which every path the change description of `pr-modes-no-agent` names reaches.
The other moves of an origin branch through `updateRemoteBranch` need no such rule:
the hand-over's move in `internal/fabricengine/handover.go` moves origin only when origin's tip is an ancestor of the owed tip and leaves a diverged one to its archive tag,
and destroy's move in `internal/fabricengine/destroy.go` refuses when origin holds commits the checkout lacks, so neither moves origin past a commit this machine lacks.
After a takeover, the new owner lacks the old owner's local refs, so a reset there is refused and named; that stop is accepted,
and its way forward is the same merge through the task worktree followed by `lym fabric push`.

Bound: the rule relies on git keeping unreachable reflog entries for `gc.reflogExpireUnreachable`, 30 days by default, and reachable ones for `gc.reflogExpire`, 90 days by default (git-gc(1), Configuration).
A test in `internal/fabricengine` drives a commit held only by the code branch's reflog and pins that it counts.
Known gap: a hand commit that entered the local code branch's reflog without the branch keeping it, such as by a reset to origin and back, passes and is reset away;
a commit of lym's own whose reflog entry expired is refused and named.

Rationale: the operator decided that a refusal stops at once with one notice;
a commit no local branch or reflog of this machine reaches was not made by lym here, so resetting past it drops someone's work.

### D8 — The dobby's catch-up hands an owed push to the pusher

When the catch-up's push of a run's owed commits fails as owed, a held push lock included, the catch-up starts the pusher of the target whose push failed and starts the run:
an owed failure of the prime's records push starts the prime's pusher, and one of the pair push starts the pair's pusher.
A refusal still fails the catch-up with its way forward.
A records commit a landing entry holds (D5) is not pushed: the catch-up reads the commits held back as owed to the pair's pusher, starts that pusher and starts the run.

Rationale: a held push lock is contention, not a failure, and the push is the pusher's to finish.

### D9 — Finalize's merge-in removes ignored leftovers of a deleted directory

The removal's base is the pre-merge task head, never the merge commit's first parent, which a fast-forward merge-in makes the task head itself.
Before it merges, the merge-in pass writes the task head it read to a base file under the run's scratch directory, named through `shedrun.ScratchDir`,
and the file stays until the removal has run.
A resume takes the base from that file alone, never from an earlier merge's `task_head` or a head read on the resume,
so a pass killed between the merge-in and the merge record write still leaves its base, on a first merge-in and on a later one.
After its merge-in, and on every pass whose base file names a base whose tree holds a directory the merged tree deletes,
Finalize runs the removal before the before-the-merge-in clean check too, so a resume past the merge reaches it.
It finds each directory whose tracked files the merged tree deleted against the base.
It removes each untracked file under such a directory that the base's ignore rules ignore, and logs each removal with its path.
Any other dirty or untracked path still halts with today's reason.
A symlink or a nested repository under such a directory is never removed.

The rules are evaluated by git itself:
Finalize writes the base's per-directory ignore files for each such directory's path chain, read from the base's tree, into an empty scratch repository,
named through `internal/lymdirs` under the hub's never-tracked directory, and runs `git check-ignore --no-index -z --stdin` there with each candidate's path.
Only per-directory ignore files from the base count, never `info/exclude` or `core.excludesFile`:
the run passes `-c core.excludesFile=` naming an empty file in the scratch repository, so git reads no user or global exclude file,
and the scratch repository's `info/exclude` is written empty.
git-check-ignore(1) documents `--no-index` as checking the exclude rules without the index,
and gitignore(5) documents that a file under an excluded directory is excluded and cannot be re-included.
A test in `internal/landingshed` pins that check against a fixture holding a nested ignore file and a negation.

Bound: only files the base's rules ignored, under a directory the merge emptied of tracked files, are removed; a tracked change or a file those rules do not ignore is never touched.
Every file those rules ignore there is removed, an ignored local settings file or secret included,
which is accepted because the project is gone from the parent.

Rationale: such files are left over from a project that no longer exists, and the run has no agent left to remove them (issue #242).
Reading the rules from the base's tree makes the removal work on a resume after the merge.

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
When origin's tracking refs already hold both tips and the parent target holds no landing entry of the run (D5), it removes the pair as today.
Otherwise the row is done and leaves the pair standing.
With git or pushes off it sets no mark, reads no tracking ref and removes the pair as today.
The dobby's landed-pair teardown, `tearDownLeftover` in `internal/beamcli/dobbydeps.go`, removes a landed pair only once origin's tracking refs hold both tips
and the parent target holds no landing entry of the run (D5); until then it leaves the pair for a later tick.
While a run's landing entry stands, every other teardown of its pair refuses before any mutation, `lym fabric remove` with or without `--force` or `--remote` included.
The check sits in `internal/pairteardown`'s teardown composite, which every removal goes through,
so no removal marks the board entry done, pushes the held records commit or anything after it, or deletes an origin branch.
Its way forward names the owed parent push: wait for the pusher, or run the step the parent's refused-push record names.
A dropped landing (D5) removes the entry before any removal, so the pusher's hand-over and a later `lym fabric remove` pass the check;
a removal of a pair whose landing was dropped pushes the held records commit and any after it to no branch.

Rationale: a teardown that waits for the network holds its run on one row, and one that removes an unpushed pair loses the work on a machine switch.

### Added after Discussion (parent, 2026-11-27): Prime code branch way forward is git push

Decision: For a refused landing push of the prime's code branch, D5's and D3's way forward is plain git push in the prime's worktree, as failedpush.go pushStep names today; D5's settle rule then writes done and clears the refused record. lym fabric push stays the way forward for a beam parent. Name the verb per target in D3 and in the D3 and D5 owned-elsewhere criteria, and add a criterion that runs the prime's way forward and sees the record clear. The diverged case, where origin's parent already holds the run's work under another commit, is its own case: its way forward drops the local landing commit (git reset --keep origin/<parent>, or a rebase that drops it when other local commits stand), and its landing entry goes with no done write. A hand-over of a run drops its landing entry and pair hold; state what that leaves on the local parent and its way forward.

Rationale: Round-3 F1: lym fabric push in the prime pushes only the records side, so the record never cleared and the pusher never restarted; merging origin's parent in after a takeover lands the work twice.

### Added after Discussion (parent, 2026-11-27): Pusher checks each carried run's owner

Decision: Before each parent push the pusher reads the owner of the parent and of every run whose landing entry the push carries, and before each done write it reads that run's owner again. A run owned elsewhere is the same refusal as a parent owned elsewhere, naming lym adopt for that run, and it holds back the whole push of that parent, since one push carries every entry on it. Add a D5 criterion: a beam run landed on the prime and adopted by another machine while its push is owed gets no push and no done write.

Rationale: Round-3 F2: today's handedOverStop reads the run's own owner; moving the push to the parent's pusher must keep that guard, as PATTERN-owner-machine requires.

### Added after Discussion (parent, 2026-11-27): No teardown while a landing entry stands

Decision: While a run's landing entry stands, every teardown of its pair refuses before any mutation: lym fabric remove with and without --force or --remote, Worktree-Teardown and the dobby's teardown alike. The refusal's way forward names the owed parent push: wait for it, or run the refused record's step. Add a criterion: lym fabric remove of a pair with a standing landing entry leaves origin's board and records without done and without the held commit, at the lowest tier that fits.

Rationale: Round-3 F3: lym fabric remove marked the entry done and pushed the held records, so origin could read an ended entry whose parent lacks the run's work.

### Added after Discussion (parent, 2026-11-27): Merge-in base persisted before the merge

Decision: D9's merge-in pass persists the pre-merge task head under the run's scratch directory before it merges, in its own file named through the existing scratch path, keeps it until the removal has run, and a resume takes the base from it, never from an earlier merge's task_head or a head read after the merge. The D9 red-first criterion adds a resume after a kill between the merge-in and the merge record write, on a first merge-in and on a later one.

Rationale: Round-3 F4, open in every round: a kill between the merge commit and the record write left no usable base, and the resume halted as in issue #242.

### Added after Discussion (arbiter, 2026-11-27): Discussion review accepted at the hard cap; round 3 findings answered in the record

Decision: The round 3 Discussion review findings are answered by the record's round 3 text, which the plan writer and Plan-Review take as settled: F1 by D5's refusal paragraph (git push in the prime's worktree for the prime's code branch, lym fabric push for a beam parent), the drop of a landing whose work origin's parent already holds under another commit, and the D3 and D5 criteria; F2 by D5's owner reads of the parent and of every run a parent push carries, again before each done write, and the PATTERN-owner-machine constraint; F3 by D13's refusal of every other teardown of a pair while its run's landing entry stands, checked in internal/pairteardown's teardown composite; F4 by D9's base file under shedrun.ScratchDir, written before the merge-in, and its kill criterion. A pull-request landing makes D5's owed-landing-push check again after it takes the parent lock and, finding one owed, releases the lock and stops as git-transport, so a local landing recorded while it waited for the lock never leaves GitHub merging onto a stale base.

Rationale: The review segment stopped at its hard cap with these four design findings open; each fix text was read against the code (failedpush.go pushStep, pushanchored.go, finalize_owner.go adopterFirstRepair, finalize.go handedOverStop, pairteardown RemovePair and HandOverOwnedElsewhere, shedrun.ScratchDir) and answers its finding. The re-check under the parent lock follows from D5's own rule and Q5, that GitHub never merges into a stale base; the round 3 review named that gap among its unraised observations.

## Constraints

- Root cause, from the code at `3fbf277ca`:
  - Issue #238: the detached pushes in `internal/fabricengine/spawn.go` and `internal/boardengine/spawn.go` record each failure through `internal/fabricengine/failedpush.go`, whose notifier, `internal/orchcli/pushnotifier.go`, mails per failure episode and per branch; they do not go through `internal/runpusher`.
  - Issues #243 and #240: `internal/sheddriver/driverules.go` re-steps a network stop after 15 s, 30 s and 1 min, then hands back.
    `internal/landingshed/finalize_owner.go` halts on an unreachable origin.
    `internal/landingshed/finalize.go` pushes the parent and the records synchronously after the merge.
  - Push lock busy: `internal/battencli/catchup.go` pushes owed commits with a 30 s lock wait and fails the catch-up on any error.
  - Issue #242: the clean-tree check in `internal/landingshed/verifygate.go` counts every untracked path, and nothing removes ignored leftovers.
  - Hand commits: `resetUnnamedTip` force-resets with a lease at the tip it read after a fetch, which a hand commit satisfies.
- PATTERN-pair-remote: the records side goes first and the code side only once it landed, for every target D1 adds that has both.
- PATTERN-landing: the local parent merge stays inside Finalize under `internal/parentlock`'s lock; only the push leaves it.
- PATTERN-owner-machine: the pusher's run-scope origin reconcile before each push stays, and so do Finalize's owner reads before its merge (D5);
  the parent target's pusher reads the parent's owner and the owner of every run whose landing entry the push carries before each parent push,
  and that run's owner again before each `done` write, the landing target from a Bolt it pulls from origin first (D5).
- PATTERN-daemon-wakeups: the pusher's wait and the driver's network wait are listed unbounded waits; a test injects their clock and never waits out an interval.
- PATTERN-config-key-names: every message names the new keys in full through `configengine`'s key-name helper.
- PATTERN-tolerant-records: the ledger refuses no unknown key.
- PATTERN-lym-files: the ledger's files and lock and D9's scratch repository are named through `internal/lymdirs` and `internal/lock`.

## Auto-mode assumptions

- The ledger, the bar field and the summary notice cover one hub, not every hub on the machine,
  since each hub has its own origin and orch; the entry's "this machine" is read as this machine's hub.
- The prime's dobby runs whenever the hub has work, so it can carry the summary notices.
- The notice threshold is one key, `fabric.push_notice_after_min`, which also governs when a waiting run enters the summary.
- The driver's network wait starts at 15 s, today's first wait, and shares the pusher's ceiling.

## Open risks

- A dobby catch-up whose fetch fails on an unreachable origin still stops that run's start (Scope, Out), although the entry says a network outage never stops other work; the parent decides whether that stays.
- A landing whose parent push stays owed holds every later pull-request landing on that parent (D5); D3's summary is its only signal.

## Acceptance criteria

- D1: red-first test in `internal/fabricengine`, tier1, for issue #238: an owed failure of `lym fabric sync`'s push mails the orch on the merge base,
  and after the change it starts the pusher, writes no failed-push record and posts no notice.
- D1: a test pins that a commit through `lym board sync`, `lym fabric sync` and `lym fabric add` starts or wakes the pusher for its target and spawns no detached push of its own; tier1 or the lowest tier that fits.
- D1: a test pins that a pair target is owed for a code commit made with plain `git commit` past the last transition's named tip, and that the prime's code branch is pushed only to a tip a landing recorded, fast-forward only.
- D1: a test pins that a landing onto a beam parent records its landing entry on the beam pair's target and creates no landing target.
- D2: a test with an injected clock pins the interval in force: 60 s after a landed push, doubled after each failure to 300 s, back to 60 s after a push lands,
  kept across two pusher processes, skipped once by a push-now mark, no extra push attempt within one interval after a transport failure,
  and a sub-second value floored with one warning.
- D3: a test with an injected clock pins that the dobby posts one summary listing every owed branch and every waiting run once a target has stayed owed for the threshold,
  one summary for a run that kept a wait record for the threshold with an empty ledger, no second summary in the same owed period, and one all-clear once the ledger empties.
- D3: a test pins that an owed push writes no failed-push record and posts no notice of its own, and that a removed pair's entry goes.
- D3: a test pins that a refused target keeps its entry and reads as refused while its refused-push record stands, keeps the bar's minutes counting and blocks the all-clear,
  and that after the record's way forward clears it, `lym fabric push` for a pair and `git push` in the prime's worktree for the prime's code branch,
  the next dobby tick drops the entry and the all-clear posts.
- D4: a test pins `lym fabric sync-status`'s output, `sync: <m>m · <s>s`, for an empty ledger and for owed targets with different intervals.
- D4, Review: `internal/reedengine/bar.go` renders the field in every hub session bar.
- D5: a test pins that a local landing whose owner reads origin answers merges under the parent lock, records its landing entry, starts the pusher
  and returns done while every push to origin fails as owed, and that a landing whose origin is unreachable stops as `git-transport` at its owner check.
- D5: a test pins that a resume reads the run as past its merge, both after a recorded landing and after a crash between the merge and the record, and that the resume records the owed tip.
- D5: a test pins that a second local landing on a parent whose first landing push is owed merges and adds its own landing entry,
  and that both runs' `done` writes are made and both pair holds released once the newer tip lands, and neither before.
- D5: a test pins that the board `done` write is made only after the parent push landed, that the run's records commit reaches origin only after it,
  that a catch-up during an owed landing push leaves Finalize's records commit and the board `done` off origin and Worktree-Teardown leaves the pair standing,
  and that a landing entry whose owed tip origin's parent already holds is settled with its `done` write and its refused-push record cleared.
- D5: a test pins that with the prime owned by another machine the landing target pushes nothing, makes no `done` write,
  writes one refused-push record whose notice names `lym adopt root`, then `git push` in the prime's worktree, and that later dobby ticks start no pusher for it.
- D5: a test pins that for a prime parent the refusal's way forward, `git push` in the prime's worktree, clears the refused-push record once it lands
  and makes the `done` write.
- D5: a test pins that a beam run landed on the prime and adopted by another machine while its landing push is owed gets no parent push and no `done` write,
  and that the refusal names `lym adopt` of that run.
- D5: a test pins that a landing whose work origin's parent already holds under another commit is dropped with no parent push and no `done` write,
  that for a run owned elsewhere the pusher hands it over and no branch on origin receives the held records commit,
  and that with a newer landing standing on it the push is refused naming a rebase, after which the next pass pushes and settles the newer entry.
- D5: a test pins that a refused landing push reads as refused in D3's summary through the prime's code-branch record.
- D5: a test in `internal/battencli` or `internal/battenshed`, tier1, pins that a batten run whose landing push is owed passes Land with origin reachable,
  and that with origin unreachable Land stops as `git-transport` instead of Stuck, both with the landing push owed and with it landed.
- D5: a test pins that with git or pushes off a local landing writes no landing entry and makes its `done` write itself.
- D5: a test pins that a pull-request landing whose parent owes a landing push stops as `git-transport` before taking the parent lock,
  and that one whose landing push was refused is a blocked halt naming the refusal's way forward.
- D6: red-first test in `internal/sheddriver`: a `git-transport` stop whose repair count is past three is still re-stepped with a wait capped at `fabric.push_ceiling_s`, where the merge base hands it back; tier1.
- D6: a test pins that a step the Go driver runs writes no halt note for a `git-transport` or `github-api` stop, that the dobby's run watch mails nothing while a wait record stands under a held driver lock,
  and that the run watch reports one stop once the driver lock is free.
- D6: a test pins that Finalize's owner check and `syncParent`'s fetched tips read on an unreachable origin are `git-transport` stops, not a blocked halt or a plain failure.
- D6, Review: the doc comment of `loomengine.AnomalyTransientStop`, the `loomcli` package doc and the reflection template name only `git-transport` and `github-api` as re-stepped with waits.
- D7: red-first test in `internal/fabricengine`: a rejected code push whose origin holds a fetched commit no local branch or reflog reaches is refused,
  naming that commit and the way forward, where the merge base resets past it.
- D7: a test pins that the pusher stops at once with one record and one notice on an authentication refusal.
- D8: red-first test in `internal/battencli`: a catch-up whose push lock stays held starts the pusher of the target whose push failed and the run, where the merge base fails the catch-up.
- D9: red-first test in `internal/landingshed`: a merge-in that deletes a tracked directory holding ignored files removes them and goes on,
  also on a resume after the merge, after a fast-forward merge-in, on a task's first merge-in with no earlier merge record,
  and on a resume after a kill between the merge-in and the merge record write, on a first merge-in and on a later one, where the merge base halts;
  a tracked change still halts.
- D10, Review: both engines' edit directives say a failed push needs no action because lym pushes the commits later.
- D11: the tests D11 names exist, each pinning behavior no other test covers.
- D12, Review: the change description names issues #218 and #254 as settled by `async-run-pusher` and says #254's bound is replaced by the quiet retry.
- D13: a test pins that Worktree-Teardown with an unreachable origin ends done with the pair standing,
  that it also leaves the pair standing while both tips are on origin but the run's landing entry stands,
  and that the dobby's landed-pair teardown removes the pair only once origin holds both tips and the run's landing entry is gone.
- D13: a test pins that `lym fabric remove` of a pair whose run's landing entry stands, with and without `--force` and `--remote`, refuses before any mutation,
  leaving origin's board without the `done` write and origin's records without the held commit; tier1 or the lowest tier that fits.

## PATTERN changes

- `PATTERN-pair-remote` — changes: Detached pushes of lym's branches: one background pusher per target, records before code, owed pushes retried quietly; a push whose result its caller reads stays synchronous; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.

## Notes for the plan writer

- The backgrounds of PATTERN-pair-remote, PATTERN-run-fsm, PATTERN-landing and PATTERN-daemon-wakeups describe the pusher's targets, the landing's synchronous pushes and the pusher's one-minute ceiling; each needs its background edit.
- The driver's network rule sits in `internal/sheddriver/driverules.go`, and its wait is applied in `internal/sheddriver/drive.go`.
- The Slack toggle in `internal/reedengine/slacktoggle.go` is the model for the bar's `#()` job.
- The run reader's past-the-merge read is in `internal/pairteardown/runread.go`.
