# Background pushes retry quietly; landing pushes and reads retry instead of failing

`push-retry-not-halt` · Task · bug, fabric, shed, landing, batten, dobby · running · Code-Bouncer

Decision record: [Sketch](decision-push-retry-not-halt.md)

## Decisions (operator, 2026-11-26)

- No mail per failed push. lym retries quietly with backoff.
  One summary notice goes to the orch when owed work has stayed unpushed past a threshold (30 minutes proposed), listing every branch still owed, and one notice when all is pushed again.
- The reed status bar shows one field, `sync: <minutes>`: the minutes since this machine last had nothing owed to origin, so the operator sees before switching machines whether work is still only local.
- Agents may still push, but a push of theirs that fails never stops them: their instructions say a failed push needs no action, since lym pushes their commits later, and no row or gate requires an agent's commits to be on origin.
- lym pushes everything it owns through the one pusher with these rules: both branches of every pair, beam branches and the board. Branches the operator made outside lym are left alone.
- How often lym pushes is config in seconds, adaptive: a short interval while pushes go through (60 s proposed), doubled after each failed push up to a ceiling (300 s to start with), and back to the short one after a push lands.
  The reed field shows the interval in force beside the minutes since the last sync, such as `sync: 3m · 60s`.
- A landing makes lym push at once, but a network outage never stops other work: other runs go on committing locally.
  A landing that needs no GitHub call merges locally and hands its owed push to the pusher; one that needs GitHub, such as a pull request, waits for the network and retries, and never fails on a transient error.
- A refusal that retrying cannot fix (rejected push, expired login, moved branch) stops at once with one notice.

## Operator intent (2026-11-26)

LoomYard stops too often because a push to origin fails, above all on VPN at work.
lym and the agents only commit; lym pushes in the background when it suits and retries a failed push later, with fewer pushes and less traffic.
A finished task is the exception: its landing is pushed at once, so a milestone never stays only local.

## What `async-run-pusher` (`b4236816a`) already built

Run transitions commit locally, and a per-worktree pusher started by each commit, the dobby's tick and a resume pushes both sides, retries owed pushes with backoff and posts one notice after a minute.
Read first what each part below still needs on top of it:
`lym fabric commit`'s and `lym board sync`'s own detached pushes, the synchronous landing pushes and GitHub reads, and Seed-IDE's check of the code tip on origin.
A part the pusher already settles is named as such in the change description and dropped.

Bundled from the notes below; each part keeps its note's text.

# Periodic async push of pairs whatever made the commits

Imported from [issue #247](https://github.com/Knatte18/loomyard/issues/247).

## Idea

Agents should only have to commit; they should never have to think about pushing.
lym takes care of every push itself, asynchronously, without blocking an agent.

## Seen

lym already pushes asynchronously after `lym fabric commit`, but only then.
An agent that commits with plain `git commit` leaves its commits unpushed until some later lym step pushes the pair, and the push failures that follow land on whichever step happens to push next.

## Proposal

A periodic pusher in lym, for example in the dobby or its own small daemon, that pushes each live pair's code and records branches when they are ahead of origin, however the commits were made.

- Periodic and not aggressive: a fixed interval of minutes, not a push per commit.
- Never blocks an agent or a row; a failed push is retried at the next period, with backoff, and reported once rather than per attempt.
- Rows that need origin current, such as Finalize, still push synchronously as now.


# Transient push failures are mailed to the orch one by one instead of retried

Imported from [issue #238](https://github.com/Knatte18/loomyard/issues/238).

On a network that resets HTTPS connections now and then (git exits 128 with 'Recv failure: Connection reset by peer'), every failed background push sends the orch a creel message naming `lym fabric push` or `lym board sync` as the way forward. In one hour on one hub the orch got eight such messages, for the bolt branch main, task code branches and records branches, and each retry by hand succeeded at once or failed again only because the connection was still flaky.

Expected: fabric retries a transport failure itself with backoff, and mails the orch only once retries are exhausted. One message then covers every branch still unpushed, rather than one message per push. The next pair push already carries a withheld code side, so a single summary would suffice.

Related: #193 (closed), which made runs retry transient fetch failures; pushes have no such retry.

# Unreachable origin halts the run as failed repeatedly instead of one retryable stop

Imported from [issue #243](https://github.com/Knatte18/loomyard/issues/243).

An unreachable origin halted the run as `failed` with anomaly `producer-hard-failure` seven times in a row, across PR-Review and Finalize, instead of waiting or blocking once.

What happened:
- PR-Review and Finalize halted with `landingshed: ... push the task branch ... fabricengine: ls-remote of <task>-weft ... Recv failure: Connection reset by peer`.
- Finalize then halted three more times with `landingshed: Finalize: read who owns the run: board: origin is unreachable, so the owner write was not pushed: fetching origin for the board ... failed (gitrepo: ... git exited 128 ...)`.
- One halt in between was `blocked` with anomaly `escalation-to-human`: `owner check: origin was unreachable, so who owns this run could not be judged; re-run "lym loom resume" in the task worktree once origin answers`.
- Each halt wrote a note and a history entry; the history shows `Finalize / stuck` then `Finalize / goto` before each re-failure.

Why it is abnormal:
- A transient network failure is classed as a hard failure, the same class as a producer defect, and the run state becomes `failed`.
- The same condition surfaced under three different halt shapes (`failed`, `blocked`, `failed`), so the operator cannot tell one outage from three problems.
- Each re-step re-ran immediately and failed again within about a minute, so the retries consumed attempts without waiting for the remote.

What to do:
- Classify unreachable-origin errors (ls-remote, fetch exit 128, connection reset) as one retryable or `blocked` kind with a single note, and have the driver wait or back off before re-stepping.
- Keep `failed` for errors that retrying cannot fix.

Provenance: task `retire-d9`; notes loom-halt-3.md, loom-halt-4.md, loom-halt-5.md, loom-halt-6.md, loom-halt-7.md, loom-halt-8.md, loom-halt-9.md.


# Finalize halts as hard failure on a GitHub API EOF that lym marks transient

Imported from [issue #240](https://github.com/Knatte18/loomyard/issues/240).

A `lym loom step` at the Finalize producer read a pull request's status from the GitHub API and got `unexpected EOF`.
The step was refused with `transient=github-api`, yet the run halted as `failed` with anomaly `producer-hard-failure`.

Quoted from the trace:

```
level=WARN msg="shed: step refused" kind=producer transient=github-api error="landingshed: Finalize: read the pull request's status: pullrequest: read pull request #54: Get \"https://api.github.com/repos/<owner>/<repo>/pulls/54\": unexpected EOF"
```

Halt note text: `loom halted failed at Finalize`, `state: failed`, `anomaly: producer-hard-failure`, 15 history entries, all earlier steps done or skipped.

This is abnormal because lym itself labels the error transient, and a single dropped connection to the API then ends a run that had finished every earlier step.
Other network failures in the same run (an unreachable origin before a push) were retried until they landed, so the Finalize read is inconsistent with them.

Suggested fix: when a step is refused with a non-empty `transient` kind, retry it with backoff, or leave the run resumable, instead of recording `producer-hard-failure`.
A failure of this kind should reach `failed` only after the retry budget is spent.

Provenance: task `transient-tripout-overpull`, from `loom-halt-2.md`.


# Seed-IDE fails the run when the task branch's detached push has not landed yet

Imported from [issue #218](https://github.com/Knatte18/loomyard/issues/218).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `.lym/logs`.

## What happened

Run `weekly-time-report` (recipe raddle, promoted from a Sketch) stopped at Seed-IDE with:

> run "weekly-time-report" cannot act: the code tip e7b0cc2 the run's last transition names is not on origin yet; way forward: run "lym batten run weekly-time-report" again once the code push lands

Timeline from `.lym/logs` (2026-11-24, CET):

- 14:45:52 Worktree-Create done; the task branch forks from the prime's local `main` at e7b0cc2, one commit ahead of `origin/main`.
- 14:45:52 and 14:45:55 fabricengine spawned detached pushes of the new pair (`fabric --warp-path …/weekly-time-report …`).
- 14:45:57 Seed-IDE's bootstrap checked whether e7b0cc2 is on origin, found it was not, and refused the step.
- Shortly after, the detached push landed: `git ls-remote origin` showed `refs/heads/weekly-time-report` at e7b0cc2.

The network was slow and resetting that day (`Recv failure: Connection reset by peer` on records pushes), so the detached push took longer than the 2–5 s gap.

## Effect

- The run stopped as failed, and the tackler handed it back to the parent asking whether to push the prime's `main`, which was not needed.
- A plain `lym batten run weekly-time-report` afterwards went straight through Seed-IDE to Run-Shed.

## Expected

A check that the code tip is on origin, right after lym itself spawned the push that puts it there, should wait for that push, or retry for a while, instead of failing the run.
The stop should also say that lym's own push of the task branch is in flight, so the tackler does not ask the parent to push the parent branch.


# Finalize blocks on untracked leftovers of a project the merge-in deleted

Imported from [issue #242](https://github.com/Knatte18/loomyard/issues/242).

## Seen

A darn run's Finalize merged its parent beam into the task worktree after its pull request had merged on GitHub.
The beam had just taken in a main commit that deleted a whole C# project.
The task worktree still held that project's directory with only untracked, gitignored content: `bin/`, `obj/` and generated fixtures.
Once git removed the tracked files, the directory was no longer ignored by the deleted `.gitignore`, so it showed as untracked.
Finalize then blocked: "the task worktree has uncommitted changes after the merge-in: …/NORCE.Import.CommonLibHarness/; commit or remove them on the task branch, then resume".

The run had no live agent left to remove it.
The orch may not edit a task worktree, and the Go driver only asked the parent a question and stopped again on resume.
The only ways forward were a goto back to the writer, which would re-run review and PR rows for a merged pull request, or the operator removing the directory by hand.

## Workaround

The operator removes the leftover directory by hand, then the orch runs `lym loom resume` in the task worktree.

## Expected

Finalize's merge-in treats a directory that holds only files the merge's base ignored, under a path the merge deleted, as leftover output:
it removes it, or ignores it in the clean-tree check, and goes on.
Failing that, the stop names a lym verb the parent can run to clear it, instead of needing a hand edit in the task worktree.



# Seen 2026-11-26: a busy push lock fails a dobby start

With several tasks released at once, dobby's catch-up for `sandbox-closed-area-build` and `keep-landed-records` failed with `fabricengine: push lock busy: .../loomyard-weft/.weft/fabric.push.lock still held after 30s ... neither side was pushed`, and each needed a manual `lym batten run`.
A held push lock is contention, not a failure: the catch-up should hand the owed push to the pusher and start the run.

# Review findings carried over from `async-run-pusher` (`b4236816a`)

Code-Review left these MEDIUM findings open; check each and close what is still open:

- tests of the pusher's lock and start conditions (`internal/runpusher`);
- tests of the pusher restart calls from the dobby tick and the batten and loom resumes (`internal/beamcli`, `internal/battencli`, `internal/loomcli`);
- tests of the teardown's push and level wiring (`teardownPushSeams` in `internal/battencli/wire.go`);
- Worktree-Teardown looping with pushes off (`internal/battenshed/teardown.go`).


## From issue #254

Imported from [issue #254](https://github.com/Knatte18/loomyard/issues/254).

## What happened

A `lym shed` run halted `failed` with anomaly `interrupted-stop` at producer `Webster`.
The note says a machine stop likely ended the loop, which is not itself a defect.
The stderr it carries shows two things that are:

1. The step wrote `status push is owed; retrying until it lands` for `producer=Drift-Expect` while origin was unreachable (`ls-remote` failed with `Connection reset by peer`).
   It then logged `origin is unreachable before a push; pushing unchecked, so the push's own failure is recorded` twice, about 10 s apart.
   The loop never produced an envelope, and the halt note was written only about 20 hours after the loop started, after the machine rebooted.
   Nothing in the notes shows a bound on the retry, or any record of the push's own failure after the "pushing unchecked" lines.
2. The note says step `Drift-Await` was in flight, while the stderr's producer is `Drift-Expect` and the history shows both `Drift-Await` and `Drift-Expect` as `skipped`.
   The in-flight step name and the producer that owed the push disagree.

Quoted lines (paths removed):

```
level=WARN msg="loomcli: status push is owed; retrying until it lands" producer=Drift-Expect state=running error="fabricengine: push code side ...: ls-remote of <branch> ...: exit 128: fatal: unable to access '<origin>': Recv failure: Connection reset by peer; way forward: run `lym fabric push` ... to retry the push"
level=WARN msg="loomcli: origin is unreachable before a push; pushing unchecked, so the push's own failure is recorded"
level=WARN msg="loomcli: origin is unreachable before a push; pushing unchecked, so the push's own failure is recorded"
```

## Why it is abnormal

A transient network failure on a status push should either land after a bounded retry or fail the step with a recorded push error.
An unbounded wait with no outcome stalls the whole run until something external kills it, and the halt then reads as a machine stop instead of a push problem.

## What to do

- Bound the "push is owed" retry, and record the push's final failure in the step's status and trace when it gives up.
- Make the halt note's in-flight step and the producer owing the push name the same step, or say why they differ.

Provenance: task `models-bit-rock-formation`, from `loom-halt-2.md`.

# A rejected push can force origin past a commit made by hand on GitHub

Folded from note B1 (board tidy, 2026-11-27).

`pr-modes-no-agent` landed with an open point in its change description: a push can still drop a hand commit made on GitHub.
For a task pair, a rejected push is retried by `resetUnnamedTip` (`internal/fabricengine/pushclassified.go`),
which can force-reset origin's code branch past the hand commit.
The change description names five other push paths that can push over such a commit.
This is read from the code and was not tested against a remote.

A push that finds origin ahead with a commit lym did not make stops and names that commit, rather than resetting past it,
as the decision above on refusals that retrying cannot fix says; check every push path the change description names.

# Left open by `transient-stop-not-failed` (orch, 2026-11-27)

That task kept state `failed` for a step failure with a non-empty transient class and gave it its own anomaly kind, `AnomalyTransientStop` (`internal/loomengine/anomaly.go`);
recording it as a retryable stop is this task's (its Code-Review finding F1, accepted unconverged).
Its finding F2 was also left open: the kind's doc comment, the `loomcli` package doc and the reflection template call every transient-stop halt one the driver re-steps,
while the driver re-steps only `git-transport` and `github-api`; `agent-start` and `host-resources` get the loop's one re-step and then go to the tackler.
