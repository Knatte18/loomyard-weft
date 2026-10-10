# Decision record: publish-recovery

## Goal

A loom run that stops at Publish or Finalize recovers through one named lyx route, never through a Plan re-run or a hand repair of the pair:

- A Publish blocked on a failing verify names the route that hands its failure to a fixer round (#462), so the hub never falls back to `goto Plan-Bouncer` for a fix the plan does not change.
- A Publish or Finalize re-stepped or resumed over the staged merge-in its own earlier attempt left behind redoes the merge-in instead of refusing on a dirty tree (#471), and the dirty-tree refusal names `lyx fabric merge --abort` when a merge is in progress.
- Finalize warns of a per-worktree config change only when the task's copy differs from the parent's, never when the change came from the parent (#450).
- A loom re-step is not refused while a run dir of a strand lyx itself stopped is younger than the attach probe's age bound (#452).
- `lyx loom decision add` refuses a `--by parent` or `--by operator` label from a lyx-spawned task session, so a plan agent's scope call reaches the parent instead of reading as the parent's decision (#453).

## Scope

In:

- Publish's Stuck reason for a failed or timed-out plan verify or `publish_verify`, the matching refusal-spec row, and the orch role's escalation list, each naming the fix route.
- The pre-merge step of Publish and of Finalize's merge-in: abort of the run's own parked fabric merge-in, and a merge-in-progress refusal for any other merge state.
- The mid-merge refusal of `lyx loom resume` and `lyx loom start`, narrowed for a run halted at Publish or Finalize over its own parked merge-in.
- The config-change notice Finalize computes: a three-way comparison instead of a fork-point diff.
- A test pinning the attach probe's disposition of a young run dir whose record reads stopped, and the burler half probe over it.
- `lyx loom decision add`'s caller check on the strand name, its refusal-spec row, its help and package docs, and the plan stencil's instruction about scope additions.
- Docs in the same commit: `internal/landingshed`, `internal/loomcli` and `internal/fabricengine` package docs where behavior changes, `contracts/specs/refusal-spec.md`, `docs/overview.md`'s verb lines for `resume` and `decision add`.

Out:

- An automatic Go-side route from a blocked Publish to the fixer round.
  Publish's Stuck covers merge, push and dirty-tree faults that stay a human's, and shed routes one `on_stuck` per row with no outcome-dependent edge; adding one is an engine change this task does not make.
- `lyx loom reject` at a blocked Publish.
  A reject needs an open pull request, which a Publish blocked before its push never has, and PR-Rework re-plans a whole generation for one fix.
- Post-Discussion parent decisions as a focus of Webster-Review.
  A decision that changes what the code must do changes the plan, and the Plan route carries it; `webster-replan-state` owns the webster state across such a goto.
- Continuing a half-resolved merge-in instead of redoing it.
  The resolver's contract is to abort any in-progress merge before a clean attempt and to re-verify a resolution by content, and this task keeps it.
- A transient class for the attach probe's young-run-dir refusal.
  The driver's transient re-step is immediate, so it would meet the same refusal; the stopped record is the fix.
- Changes to the driver stencil's repair rules.
  With the pre-merge abort the re-step is no longer a guaranteed refusal, so the driver needs no merge awareness.
- The Webster-Burler round gate's tiers and the Publish failure record's shape, which `gate-test-scope` settled.

## Decisions

### D1. Publish's verify failure names the fix route

Decision: when Publish's plan verify or `publish_verify` fails, times out or cannot start, its Stuck reason ends with the way forward: run `lyx loom goto --to Webster-Burler`, then `lyx loom resume`, in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests.
The refusal-spec gains a `Publish: verify failed` row carrying the same way forward, and the orch role's "Escalations from a child" list gains the item: a Publish blocked on a verify failure takes that goto, never `Plan-Bouncer`.
The Stuck reason for a dirty tree is unchanged.

Rationale: `gate-test-scope` built the receiving side, the failure record, the rubric's `## Publish failure` section and the round gate's failing-test steps, but nothing names the route, so the hub keeps choosing between a plan re-run and a hand fix.
`Webster-Burler` is the admitted goto target closest to the fixer: its reviewer half reads the rubric with the failure section and raises each failing test as BLOCKING, its fixer half fixes on source, its verify gate confirms the fix, and its `on_stuck` hands to Webster-Bouncer's judge, then Describe and Publish.
A goto to Webster-Bouncer first spends a judge turn on findings the record already states.

Bound: the goto is the hub's command, one per failure; a goto gives the Webster-Review segment a fresh bounce budget, so the loop is bounded by the hub's decision each time Publish blocks again.
Publish reruns both verifies in full on its next pass and removes the record only then, so a fix the round gate accepts still cannot ship unverified.

### D2. Publish and Finalize abort their own parked merge-in before the clean check

Decision: before the pre-merge clean check, Publish's merge-in and Finalize's merge-in step probe the pair's merge state through fabric.
A parked fabric merge record whose `Source` is the run's parent branch is the row's own leftover: the row aborts it through `MergeAbort`, logs the abort at Info with the record's source and start SHAs, and continues to the clean check and a fresh merge-in.
Any other merge state stops the row Stuck before the clean check, with a reason that names the state and its way forward instead of listing the staged files: a parked fabric merge with another source names `lyx fabric merge --continue` or `lyx fabric merge --abort`, then resume; foreign git merge state (a `MERGE_HEAD` or unmerged entries fabric did not start) names concluding or aborting it with git, then resume.
The resolver's own unconditional abort at the start of a clean attempt stays as crash recovery within a step.

Rationale: the resolver already redoes the merge-in from scratch on every call, so the clean check that runs ahead of it only turns a leftover the row would discard anyway into a false "uncommitted changes" halt whose advice, to commit or remove them, is wrong on both branches.
Reading the record's source keeps the abort to merges the row itself makes.

Bound: the abort discards a conflict session's resolution of that merge, which the resolver's contract already discards, and the redo costs one more conflict session.
A merge with another source or foreign state is never touched.

### D3. The mid-merge refusal of resume and start lets the row's own leftover through

Decision: `lyx loom resume` and `lyx loom start` keep their `merge_in_progress` refusal, except for a run halted at Publish or Finalize over a parked fabric merge whose `Source` is the run's parent branch, which they let through, since the row aborts and redoes it (D2).
The refusal for a parked merge keeps naming `lyx fabric merge-stage`, `lyx fabric merge --continue` and `lyx fabric merge --abort`; the refusal-spec row for `resume: merge in progress` states the exception.

Rationale: a blocked halt parks the driver, so the hub's resume is the path that reaches the row's re-step; a refusal there makes the hub run `lyx fabric merge --abort` by hand, the workaround #468 recorded.
The exception is as narrow as D2's abort, so nothing resumes over a merge the row would not discard.

Bound: an orch that half-resolves its own merge-in of the parent branch at a Publish halt and then resumes loses that resolution to the row's redo; the orch role already forbids editing in a task pair.
Every other row, every other source and all foreign state keep the refusal.

### D4. The config-change notice reports only files the task changed away from the parent's copy

Decision: Finalize's config-change read compares three points on the records branch: the fork point (merge-base of the parent's tip and the task's tip), the task's tip and the parent's current tip.
A per-worktree config file is reported only when its content at the task's tip differs from both the fork point's and the parent's tip.
The notice names the parent tip beside the base and tip SHAs it names today.

Rationale: a parent commit that reaches the task through a records sync differs from the fork point but equals the parent's current copy, so it drops out; a parent change the task never synced is unchanged on the task side, so it drops out; a task edit the parent later made identically needs no re-apply, so it drops out; a task edit that still differs from the parent is the one case the notice exists for.

### D5. A run dir of a strand lyx stopped is dead at once

Decision: the stop verb's `stopped` record, landed by `shuttle-send`, is the fix for #452; this task pins the incident's shape at the attach probe and the burler half probe: a candidate that is untracked or carries a cleared pane binding, whose record reads `stopped`, and whose directory is younger than the age bound, is respawn-eligible, and the burler half probe over it reports the half gone, or done when its output exists.
No code changes unless the scenario fails.

Rationale: `leftoverThenAgeVerdict` already treats a terminal outcome as respawn-eligible at any age and `stopped` is terminal, so the remaining gap is coverage of the exact path the incident took.

### D6. `lyx loom decision add` refuses a task session's label

Decision: the verb reads the strand name reed exports to a spawned session and, when it parses as a name carrying a slug, refuses both `--by parent` and `--by operator` with the way forward: the parent, `ly:orch`, or the operator runs the verb from a slug-free session; a plan agent sends its scope call to the parent by message and plans on it once the record carries it.
An empty name and a slug-free name pass.
The refusal is a `decision: task session` row in the refusal-spec, and the verb's package doc and help drop the sentence that it checks no caller identity.
The plan stencil's instruction to record a scope addition with `lyx loom decision add` is replaced by: a scope addition that arrives after the Discussion is a design call the parent records; ask `ly:orch` to record it and plan on it once the record carries it, and never write it into the plan as an operator addition.

Rationale: the parent and the operator are the two slug-free callers, and the orch's strand name is slug-free, so the same check `lyx config` makes for a hub-wide write tells them from every run agent without a new identity mechanism.
Refusing both labels keeps a task session from relabelling its call as the operator's.

Bound: the check reads the exported name and nothing else about the session, so it guards a mistaken call, not a determined agent that clears the variable.
When a driver route for recording decisions is built (`run-message-routing`), it needs its own label or exemption; none is given now.

## Constraints

- `PATTERN-refusal-way-forward`: every new Stuck reason or refusal names its way forward, lands with its refusal-spec row and a test reaches it (D1, D2, D3, D6).
- `PATTERN-no-denied-recovery`: the ways forward name `lyx fabric merge --abort`, `lyx fabric merge --continue`, `lyx loom goto`, `lyx loom resume` and plain git merge verbs, none of which is a denied form.
- `PATTERN-fabric-git` and `PATTERN-fabric-destruction-chokepoint`: the abort in D2 goes through fabric's `MergeAbort`, never raw git, and the merge-state read through fabric's probes.
- `PATTERN-mutation-record`: D2's abort runs inside a producer, not a new fabric verb; no new verb is added.
- `PATTERN-told-geometry`: the landing producers take the merge-state probe and the abort as told seams, as they take the resolver today.
- `PATTERN-agent-name`: D6 parses the strand name through `internal/agentname` only.
- `PATTERN-transient-stop`: no new transient mark; D5 adds none and D2's Stuck is a verdict.
- `PATTERN-verified-tree`: Publish's verify sites and the failure record are unchanged.
- `PATTERN-review-round` and `PATTERN-batcher-registry`: the goto route in D1 starts an ordinary Webster-Review round and touches no webster state.
- `PATTERN-sole-parsers`: the decision record is still written only through `discussionparser`.
- `PATTERN-test-economy` and `PATTERN-test-speed`: each scenario below extends an existing table where one covers the surface, and the landing producers' scenarios stay in the fake-backed tier.
- `PATTERN-comment-line-breaks`, `PATTERN-markdown-link-integrity`, `PATTERN-documentation-lifecycle`: docs and comments land in the same commit as the code.
- Planned work on `webster-replan-state` owns goto-to-Plan webster state; D1's route never re-enters Plan, so the two do not overlap.
- `driver-context` may later move the driver loop into Go; D1's route stays a hub command until then.

## Auto-mode assumptions

- The hub, not Go, runs the goto in D1; an automatic route is out of scope for the reasons given there.
- The goto target in D1 is `Webster-Burler`, not `Webster-Bouncer`.
- D2 aborts and redoes rather than continues a half-resolved merge.
- D3 narrows the resume and start guard only for Publish and Finalize over a parent-sourced parked merge.
- D4 compares against the parent's current tip on the records branch, taking the merge-base as the fork point as the current code does.
- D5 changes no code unless its scenario fails.
- D6 refuses both labels from a slug-bearing session and gives the driver no exemption.

## Open risks

- D1 still costs one hub command per blocked Publish; moving it into Go waits on an outcome-dependent routing edge nobody has designed.
- D2's redo spends a conflict session on a merge an earlier session had resolved.
- D3 can discard an orch's manual, half-resolved merge-in of the parent at a Publish halt.
- D4 assumes the records branches share history so the merge-base is the fork point; a rewritten parent records branch would make the base wrong, as it does today.
- D6 relies on an exported environment variable, which a session can clear.

## Acceptance criteria

- Publish with a failing, timed-out or unstartable plan verify or `publish_verify` stops Stuck with a reason ending in the `lyx loom goto --to Webster-Burler` then `lyx loom resume` way forward; the dirty-tree reason is unchanged.
- Both Webster-Review rows, the Bouncer judge and the Burler reviewer, render the Publish failure section from the record.
- The refusal-spec carries rows for `Publish: verify failed`, `Publish or Finalize: merge in progress`, the narrowed `resume: merge in progress`, and `decision: task session`, and a test reaches each.
- Publish re-stepped over a parked fabric merge whose source is the parent branch aborts it, logs the abort, runs the merge-in afresh and never reports the staged paths as uncommitted; Finalize's merge-in step behaves the same.
- Publish and Finalize over a parked fabric merge with another source stop Stuck naming `lyx fabric merge --continue` and `lyx fabric merge --abort` with no path list; over foreign git merge state they stop Stuck naming git.
- `lyx loom resume` and `lyx loom start` proceed for a run halted at Publish or Finalize over a parent-sourced parked merge, and keep the `merge_in_progress` refusal at every other row, for another source, and for foreign state.
- Finalize's config notice: a parent-side config change synced into the task yields no notice; a task-side edit yields one naming the file, base, tip and parent tip; a parent change the task never synced yields none; a task edit identical to the parent's current copy yields none.
- The attach probe classifies an untracked or binding-cleared candidate with a `stopped` record and a directory younger than the age bound as respawn-eligible, and the burler half probe reports such a half gone, or done when its output exists.
- `lyx loom decision add` under a slug-bearing strand name refuses both labels with the way forward and appends nothing; under an empty or slug-free name it appends as today.
- The plan stencil no longer tells the plan agent to run `lyx loom decision add`, and the orch role names the D1 route.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` are green.

## Notes for the plan writer

- The failure note reaches the rubric through `loomcli`'s wiring of `PublishFailureNote`; confirm the Burler row's reviewer prompt interpolates the same rubric marker.
- `fabricengine.MidMerge` answers parked versus foreign but not the record's source; the landing rows need the source, which the merge record carries, so either widen the probe's answer or read the record through the resolver's fabric seam.
- Finalize's `mergeInStep` runs twice on the retry after the parent moved; the abort belongs inside it so both passes behave alike.
- The resume and start guard sits in `refuseOverUnfinishedMerge` in `loomcli`; it needs the run's current producer and parent branch, both already read nearby.
- The three-way comparison lives in `fabricengine.ReadConfigChanges`; `git diff --name-only` between the parent tip and the task tip over the same pathspec gives the second set.
- Attach's disposition table in `attach_test.go` and the stop tests are the places for D5's rows; `burlerengine`'s probe has its own fakes.
- The config write refusal in `configcli/hubwide.go` is the shape D6 mirrors; the strand name comes from `agentname.StrandNameEnv`.
- The test budget rows for `internal/landingshed`, `internal/loomcli`, `internal/fabricengine` and `internal/shuttleengine` may need raising; a card that does so lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
