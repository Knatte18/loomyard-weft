# Decision record — orch-full-autonomy-mode

## Goal

An autonomy mode under which the run's parent orch receives every halt of every run under a worktree and settles it itself.
Autonomy is decided per worktree: one beam may run autonomous while another beside it does not,
the prime's root tasks may take it through the prime's own setting, and a hub may take it for every worktree.
The operator is reached only in two cases:

1. A severe halt: one of the closed classes D5 names.
2. A merge-ready beam: lym opens the beam's pull request, then the orch notifies the operator.

## Scope

In:

- A `beam.yaml` key `autonomy`, its layers, and the readers that answer it.
- What autonomy forces on a run's existing ask, approval and interactivity keys.
- PR-Review of a member task under autonomy: reviewed by the parent orch, ended only by its recorded satisfied answer.
- The orch's handling of halts under autonomy, the closed list of severe halts and the marker on a relay to the operator.
- A `[beam drained]` notice from Beam-Await, and the orch's merge-ready steps that follow it.
- The Slack reply hook forwarding only operator-marked replies under autonomy.
- Telling the orch its worktree's autonomy at every launch, and each halt's autonomy on its notice.
- Glossary lines for the two new terms.

Out:

- Turning member pull requests off (D4).
- Merging the beam's pull request outside Finalize (D7).
- Releasing draft tasks, filing tasks of the operator's own and promoting notes: these stay the operator's under autonomy.
  Under autonomy a raddle files draft tasks with no approval (D3), and the orch files a beam's merge task (D7).
- A sampler run's Sampler-Session: the operator picks that recipe to sit in it, so it still waits on the operator.
- Turning autonomy on in the Models hub on OSL-1046 or in loomyard: a config write the operator orders after landing, since a run writes no hub config.

## Decisions

### D1 — `autonomy` is a `beam.yaml` key, a Guard key

**Decision:** `beam.yaml` (`internal/dobbyengine/config.go`) gains `autonomy`, a boolean, default `false`, classified `CategoryGuard`.
It is set with `lym config beam --set autonomy=<true|false>` in a worktree, for that worktree,
or with `--layer default` from the prime, for every worktree of the hub.
A worktree layer overrides the default layer, so a beam can opt out of a hub-wide `true`.
A worktree's file is tracked under its `_lym/config/` on the worktree's weft branch,
so a worktree forked from it starts with the file as it stood at the fork point:
`fabricengine.ReadConfigChanges` reads a task's config files at the merge-base of the task's and the parent's weft branches,
and its row "a file present at the fork point carries its base bytes" (`internal/fabricengine/configchanges_integration_test.go`) pins it.
So a beam or nested beam made under an autonomous worktree starts autonomous until its own file is set `false`.
Finalize's config-change notice (`internal/landingshed/finalize.go`) leaves `autonomy` out,
so a landing never asks an orch to re-apply a child's autonomy on its parent.

**Rationale:** `beam.yaml` is the existing per-worktree module whose `pull_request` a member's landing already reads from its parent worktree;
projects carry no `run_settings`.
Autonomy lets the parent's answer end PR-Review in the operator's place, which hollows an approval gate, so PATTERN-config-layers makes it a Guard key.
As a Guard key the local layer refuses it, so a task's `run_settings` can never make a run autonomous: only a tracked worktree or default file can.
A worktree's autonomy is its own, so it never travels upward on a landing.

### D2 — Whose value applies: a halt carries its run's value, an orch's own steps read its worktree's

**Decision:** two readers answer autonomy, both through `dobbyengine.LoadConfig`, both at call time, never from a value captured at wiring;
the one exception is PR-Review, which records the run's value once per review (D4):

- A run's autonomy is its parent worktree's resolved value.
  A root task's parent worktree is the prime; a beam's own pawl run's parent worktree is the worktree the beam was made from.
- An orch's autonomy, for the steps it takes for its own worktree (D6, D7, D8), is its own worktree's resolved value.

Every halt notice lym posts for a run carries `autonomy: <true|false>`, the run's value read by the run reader when the notice is built.
An orch handles a halt that reaches it under the value the notice carries, never its own worktree's, a beam's own Finalize halt included.
A stack that fails to load reads as `false` and is logged as a warning naming the worktree.

**Bound:** a run's notices reach the nearest live orch at or above its parent worktree, else the prime's
(`orchcli.ParentStrand`, through `orchroute.FromPair`), which need not be the parent worktree's own orch.
A halt reaching an ancestor or the prime orch is still handled under the run's value:
a run that resolves `false` is relayed to the operator by an orch whose own worktree resolves `true`,
and a run that resolves `true` is settled by an orch whose own worktree resolves `false`.

**Rationale:** the value on the notice keeps a worktree's opt-out in force whichever orch its halts reach when its own orch is down.
Reading `false` on a failure falls back to the operator being asked, never to a skipped operator.

### D3 — Autonomy forces the parent asks on and every operator-seat session off

**Decision:** under a run's autonomy (D2) the run reads these keys as forced, whatever its own layers say:

- `pr_review_ask_parent` (loom) reads `true`.
- `judge_ask_parent` (drift) reads `true`.
- `judge_interactive` (drift) reads `false`.
- `discussion_interactive` (loom) reads `false`.
- `raddle_auto_approve` (loom) reads `true`.

The forcing is applied to the module's resolved config, before any reader uses a field, so a key read in several places takes one value.
It is applied at the run's setting lookup as well,
since `raddle_auto_approve` is read there (`internal/loomshed/raddlewrite.go`) and a profile's raddle block may set it.
No layer file is rewritten, so turning autonomy off restores each key's own value at its next load,
with one exception: Raddle-Write records the run's mode in its state on its first call and reads it from there after,
so a change of autonomy reaches only a raddle run not yet at Raddle-Write.
`code_review_ask_parent` and `discussion_review_ask_parent` keep their own values:
this reads the entry's "sets the existing ask keys" as the keys that move an operator wait to the parent.

**Bound:** with `judge_interactive` forced off, no operator decides the judge's rulings, and only the parent agrees to them.
Under autonomy a judge ask that ends without the parent's answer stops the run at Drift-Verify,
a halt to the orch, instead of letting the rulings stand as written.
That covers each end state the parent-agreement gate (`internal/driftengine/judgeparent.go`) passes on today:
timeout, an ask the parent leaves unanswered past `judge_parent_timeout_min`;
not-sent, an ask the judge's own timeout leaves the parent no time for;
and no-parent, an ask that never reaches the parent because no parent orch resolves or its post fails.
The halt names `lym loom resume` and the end state.
The resume of Drift-Verify clears the ended ask, so the gate sends a new ask over the same rulings,
with a deadline counted from the resume; this re-ask is new, since today a new ask goes out only for new rulings.
A cause that persists, such as a parent orch still down, halts again, and a third halt at the row is D5's class 5.
A does-not-match ruling still stops the run, as today.
Without autonomy each of those end states lets the rulings stand, as today.

**Rationale:** the judge ask lets the orch rewrite a drift ruling before a does-not-match stops the run, the largest share of halts,
and the PR-Review ask is how the parent reviews a member pull request (D4).
An interactive Discussion or judge and an unapproved raddle proposal each wait for an operator in a pane, which autonomy rules out;
a raddle filing makes draft tasks, so their release stays the operator's.
The code and discussion review asks add a veto per round but remove no operator wait.

### D4 — Under autonomy, PR-Review ends only on the parent's recorded satisfied answer

**Decision:** whether a member task opens a pull request stays `landingshed.ResolvePullRequest`'s decision, unchanged;
an operator who wants no member pull requests sets the beam's `pull_request` off.
PR-Review records the run's autonomy (D2) in the pull request record (`internal/pullrequest`) once,
when it starts a session for a record that holds no autonomy.
Every later `ensureSession` of the same review, after a pause or a judging stop, builds the session spec from the recorded value,
so the recorded value governs the review to its end, whatever later changes the layers.
`lym loom goto` onto PR-Review clears the record's autonomy and its session fields, as `settleSendBack` clears the session,
so the re-entry records the value anew and starts a new session rather than resuming the recorded one.
Goto admits that only on a run halted at PR-Review or later, never on one PR-Review still holds.
An operator who turns autonomy off while a review waits takes it over by answering its asks from their own unnamed shell,
which `lym loom parent-review answer` admits as a caller.
Under that recorded autonomy:

- The PR-Review session (`internal/landingshed/prsession.go`) starts with `Interactive` false, and its prompt renders the autonomy.
- Each review ask to the parent is an ask in burlerengine's run-wide ask record (`internal/burlerengine/parentreview.go`),
  the record the `parent-review` verbs keep for the run's Code-Review, Discussion-Review and judge asks.
  A PR-Review ask carries the tag of the `[pr-review request]` line and a seed of its own,
  and its `Digest`, the field naming content that can change under an ask, holds the task branch's head it was sent at.
  Its deadline is `pr_review_parent_timeout_min` (loom, new, default 60) from the ask.
  The row sends the review's first ask when it opens the review; its number is the run's next, since the record numbers every ask in one sequence.
  The session's agent sends each re-ask after its fixes with `lym loom parent-review ask <slug>` (new),
  which reads the head, records the ask and posts its line to the parent orch,
  then waits for the answer with `lym loom parent-review await`.
- Each `[pr-review request]` line names `lym loom parent-review answer <slug> --ask <n>` with `--satisfied` or `--findings-file` as the answer,
  in place of the `lym creel post <strand>` answer `askParentReview` (`internal/landingshed/prreview.go`) names today.
  The answer verb records as it does today: it answers the record's latest ask before that ask's deadline and writes the parent's report beside the record.
  The rows that make the run's other asks never run while PR-Review waits,
  and a PR-Review entered again after a send-back sends a new first ask,
  so the record's latest ask during a review is that review's newest PR-Review ask.
  `pr-continue` and the row read the newest PR-Review ask by its tag (`ParentAskRecord.LatestTagged`),
  so a review or judge ask of the same run is never read as one.
  An ask that reaches its deadline unanswered is set timeout, as every ask is, and the session asks again with `lym loom parent-review ask`.
  The answer verb's refusal of a PR-Review ask that has ended names waiting for the session's next `[pr-review request]` line as its way forward,
  in place of the `lym loom decision add` that `parentReviewWayDecision` names for the other asks, since no decision ends a review.
  The `parent-review` group's help and its `answer` help name the PR-Review ask beside the review and judge asks.
- `lym loom pr-continue` ends the review only when the newest PR-Review ask was sent at the task branch's current head and is done with a satisfied report.
  With a mailed head standing and no such answer, it clears the mailed head, so the row gates that head again, and leaves the review open;
  it reports that the review stays open.
  With neither, it refuses.
  While the newest ask stands unanswered at the current head, the report and the refusal name waiting for the parent's answer,
  which the hold notices raise again; otherwise they name asking the parent again with `lym loom parent-review ask`.
- The row's `judge` ends the review on the continue flag only when the newest PR-Review ask is done satisfied at the head it judges,
  that ask's head equal to the gated head.
  Otherwise the row clears the continue flag, leaves the review open,
  and mails the session that the review stays open at the new head and that it asks the parent again with `lym loom parent-review ask`.
  So a commit made after a passing `pr-continue` is gated and waits for a new ask at the new head.
- The gate-failure mail's hint (`gateRetryHint` in `internal/landingshed/prreview.go`) drops its condition that the operator has asked to continue,
  and names `lym loom pr-continue` as the agent's own way to gate the head again.
- On a closed pull request `pr-continue` is refused, its way forward naming the row's closed-without-merge halt,
  and the row's `judge` never records `ReviewEnded` there.
  The row so reaches that halt with the continue flag unset.
  Finalize (`internal/landingshed/finalize_pr.go`), on a closed pull request whose record holds autonomy `true`,
  takes its closed-without-merge halt whatever `ReviewEnded` and a merge-at-once decision say, and reopens nothing,
  so a pull request closed on GitHub after the review ended halts too.
  The orch relays either halt as a D5 class 4 severe halt.
- On a satisfied answer at the current head the session runs `lym loom pr-continue`, which ends the review.

The orch still never approves, merges, closes or comments on the pull request on GitHub.
With the recorded autonomy `false`, PR-Review and `pr-continue` behave as today.

**Bound:** only a run whose recorded autonomy is `true` lets the parent's answer stand in for the operator's,
and only a recorded satisfied answer at the head the row judges ends the review on the continue flag:
`pr-continue` checks it before it sets the flag, and the row's `judge` checks it again before it ends on the flag.
A merge or an approval of the gated head made on GitHub ends the review as today, since only a human acts there.
`lym loom parent-review ask` is refused outside a PR-Review session whose pull request record holds autonomy `true`,
and refused while the newest PR-Review ask stands unanswered at the current head;
each ask records the head it read.
PR-Review's own gate on the head and Finalize's merge gate run unchanged;
a GitHub rule that needs a human approval still refuses the merge, which is a D5 class 4 severe halt.
A waiting session's held turns raise hold notices to the parent, as an autonomous session's do today, so an unanswered ask is raised again.

**Rationale:** keeping the pull request keeps the member trail the beam's description lists and keeps PATTERN-landing's resolver the sole decider.
A recorded answer makes the parent's ending mechanical, since D1's Guard rests on it.
Recording the autonomy once per review keeps the session's interactivity, `pr-continue`'s check and the row's ending on one value.
The run's ask record already holds the lock, the caller check, the report files and the await that a PR-Review ask needs.

### D5 — The orch settles every halt under autonomy; a closed list of severe halts goes to the operator

**Decision:** halt notices keep their routing.
Under a halted run's autonomy (D2) the orch settles every halt and run-level call itself, as the role already has it,
and also settles a choice between designs the board entry leaves open, recording it with `lym loom decision add <slug> --by parent`.
It resumes a halted run on its own judgment, never waiting for the operator's word.
A drift does-not-match stop is a run-level call:
a decision and a goto that send the run back to fix the code, or `lym drift amend` when the expectation was wrong.

A severe halt is one of these classes, and no other:

1. An owner halt (`owner check:`) or any takeover of a run onto this machine.
2. A scope change: work the task's board entry rules out or lacks, or a new task the orch would have to file, since it files none beyond D7's merge task.
3. Giving a run up with `lym batten cancel`.
4. A step only the operator can take outside lym: on GitHub (a closed pull request to reopen, a human approval a branch rule requires) or on the machine (credentials, disk).
5. A repeating stop: a halt at a row of a run whose status history already holds two earlier halts at that row.
6. A halt the orch finds no settling action for: no verb open to it settles the halt.

The orch counts class 5 from the run's status history (`shedengine.HistoryEntry` in `internal/shedengine/status.go`),
which lym writes on every halt whatever verb later settles it, and which outlives the orch's context cycles.
A halt's entry is a Stuck at the row with no `routed_to`, since a bounce names the row it routed to and never folds with a halt.
Each such entry counts as one halt plus its `repeats` (`HistoryEntry.Repeats`),
since lym folds an identical consecutive halt into the previous entry and a resume writes no entry.
Each earlier halt at the row ran again, so the orch settled it.
It files the bug a repeating stop shows with `lym selfreport create` before relaying it, never as a task.
lym does not classify severity: the orch classifies each halt against this list.
Its relay opens `[operator] severe halt <slug>:`, followed by the halt, what the orch tried and the one action it asks for.

**Bound:** the list is closed; a halt outside the six classes is never relayed as severe.
Class 6 gives every halt a route: one the orch cannot settle reaches the operator.
Class 5 bounds the orch's own settlements at two per row of a run, counted from the status history's halt entries with each one's `repeats`.

**Rationale:** the role already settles most halts; the operator waits came from design choices, the resume word and PR endings.
A closed list keeps the operator's two cases checkable, and a fixed opening makes a relay recognisable in the pane and on Slack (D8).

### D6 — Beam-Await posts one `[beam drained]` line when an autonomous beam's work is done

**Decision:** Beam-Await (`internal/pawlengine/beamawait.go`), while the beam's orch's autonomy (D2) is `true`,
posts one `[beam drained] <slug>` line to the beam's orch through its existing `Notify` seam once all of these hold:
the beam holds commits beyond its parent, no member task of the beam is open, drafts included,
no child pair of the beam exists, and no ready record stands.
The posted drain is kept as a drain marker, a file of its own beside the pawl markers under the beam's `.lym/beam/` directory,
named with the other file names in `internal/pawlengine/landrequest.go`.
Beam-Await alone writes and removes it; the await marker's writers leave it alone.
A re-entry of the row and a dobby-down notice cycle post no second line;
a member that opens again removes the marker, and the next drain posts again.
The check runs on the row's existing wakes.
With autonomy `false` it posts nothing.

**Bound:** the line only informs; it opens, merges or readies nothing.
A takeover or a new pawl run starts without the marker, so it may post one further line for a drain already posted.

**Rationale:** Beam-Await already wakes and already reaches the beam's orch, so it sees a drained beam without a new poller.
Requiring commits beyond the parent keeps a new, empty beam from draining.

### D7 — On `[beam drained]`, the orch readies the pull request and notifies the operator

**Decision:** under its autonomy the beam's orch takes each drained line as its go-ahead for the beam's landing steps up to the pull request,
starting at the first step not yet done:

1. When the beam conflicts with its parent, which `lym beam pr open` refuses (`judgeOpenable` in `internal/beamcli/pr.go`),
   run `lym beam merge-parent <slug>`.
   Its merge task starts released; its landing reopens and then drains the beam, and that next drained line resumes these steps.
   A beam only behind its parent goes straight to step 2, since `lym beam pr open` passes it and GitHub merges it.
2. When no description is recorded, write it with `lym beam describe <slug> --draft`, edit it, and record it with `lym beam describe <slug>`.
   When one is recorded, add any new member to it and record it again.
3. Run `lym beam pr open <slug>`, which opens the pull request ready for review, never as draft, or finds the open one.
   A conflict refusal sends the orch to step 1.
4. Read `lym beam pr status <slug>` and notify the operator with `[operator] merge ready <slug>:`, the pull request's URL and any blocker the status names.

The operator's word to land is the go-ahead for `lym beam ready <slug>`, which the orch then runs; Finalize merges as today.

**Bound:** the go-ahead covers only these four steps for the beam the line names; `lym beam ready` still waits for the operator.
`lym beam merge-parent` is the one step that starts work the drained line licenses, and only on a conflict refusal.

**Rationale:** the orch writes the description, and PATTERN-github-auth lets only `lym beam pr` open the beam's pull request, so lym opens it through the orch's verb.
PATTERN-landing keeps the merge in Finalize, so the operator's act is the word to ready the beam.

### D8 — Under autonomy, the Slack reply hook forwards only operator-marked replies

**Decision:** while the orch's autonomy (D2), read at call time, is `true`,
the Slack reply hook (`internal/slackcli/hook.go`) forwards a turn's reply only when it opens with `[operator]`.
While the presence is slack, the orch opens every reply meant for the operator with it, whatever its autonomy,
so no reading of autonomy decides whether a reply is marked:
D5's severe-halt relay, D7's merge-ready notice and its answers to the operator's own words, typed or from `<shortname>:SLACK`.
The orch also posts each severe-halt relay and merge-ready notice to `<shortname>:SLACK` with `lym creel post`, as the presence line directs,
so the hook's filter governs turn replies alone.
A relay opens its own reply, one relay per turn.
With autonomy `false` the hook forwards every reply, as today.

**Bound:** the filter drops only replies without the marker, and only while the presence is slack; the pane shows every reply.
A relay or notice the hook drops still reaches Slack through its direct post.
A marked relay so reaches Slack twice, through the hook and the direct post: a duplicate is the price of a relay never lost.

**Rationale:** the orch's turns on settled halts would otherwise reach the operator on Slack on every notice, against the goal's two cases.

### D9 — The orch is told its autonomy at launch and reads it at call time

**Decision:** the start, resume and reload pointers, and the pending resume the watcher types after a handoff,
each carry one line naming the orch's autonomy, read then, whatever the presence.
The line is appended to each of those pointers beside the presence step, never inside it,
since `orchengine.WithPresence` (`internal/orchengine/prompt.go`) returns a pointer unchanged outside a slack presence.
The line informs only; the orch acts on the value read at call time:
a halt's value from its notice (D2), and its own worktree's value from `lym config beam --resolved`, run when it takes a D7 step.
The orch role (`contracts/orchrole/orch-role.md`) points to a new on-demand file, `contracts/orchrole/role-autonomy.md` (new),
to read while its autonomy is on or a halt notice carries `autonomy: true`.
The file holds D5's handling and severe-halt list, D4's PR-Review answers, D7's steps and D8's direct posts.
D8's marker rule holds whatever the autonomy, so it goes in `contracts/orchrole/role-slack.md`, which every orch under a slack presence reads.
These texts name the autonomy file as their exception:

- `orch-role.md`'s rules that wait for the operator's word, its working rule that a step starting work needs the operator's explicit go-ahead,
  and its "Reviewing a pull request" section, which prescribes the creel answer and says the operator ends the review.
- `role-slack.md`'s rule that forwards every reply.
- The Long help of the `lym beam pr` group and of `lym beam describe` in `internal/beamcli/cli.go`, which say the orch runs them when the operator says so.

**Rationale:** the role is rendered from embedded bytes for every orch, so a per-worktree mode reaches it as a launch fact and an on-demand file, as beams and landing halts do.
Reading at call time keeps the orch in step with D2, D3, D4 and D8 however the value changes:
a `--set` write, an editor save, a reconcile, a hand-edited committed file or a default layer pulled from another machine.

### D10 — Two glossary lines

**Decision:** `internal/heddleengine/glossary.md` gains:

- `- **autonomy**: a worktree's mode in which its orch settles every halt of the runs under it and reaches the operator only for a severe halt or a merge-ready beam. Never: autopilot, full auto.`
- `- **severe halt**: a halt in one of autonomy's closed classes, which the orch relays to the operator. Never: critical halt, blocker.`

**Rationale:** the design adds both terms, and records and notices use them.

## Constraints

- PATTERN-config-layers: `autonomy` is classified in `beam.yaml`'s `ConfigSpec` in the same change, as Guard; no reinterpretation is needed, since the key is new.
  `pr_review_parent_timeout_min` is classified in loom's `ConfigSpec` as `CategoryNone`, as its sibling parent timeouts are, and must be at least 1.
- PATTERN-landing: `landingshed.ResolvePullRequest` alone decides a member's pull request, and Finalize alone merges, under the parent lock.
- PATTERN-github-auth: the beam's pull request is opened only by `lym beam pr`, which never merges; the PR-Review session runs no `gh` of its own.
- PATTERN-daemon-wakeups: D6 rides Beam-Await's existing wakes, which back off to one minute.
- PATTERN-agent-addressing: every new line goes through creel, through the seams that post today.
- PATTERN-refusal-way-forward: D4's `pr-continue` refusals, the closed pull request's included, its `parent-review ask` refusals and D3's ended-ask halt name their way forward, and a test reaches each.
- PATTERN-stencil-ownership: `role-autonomy.md` and the pointer line render through the orch role's existing path, `orchengine.RenderRoleFile`.
- The resume, decision, circling, drift-amend, selfreport and beam verbs D5 and D7 name are open to an orch session,
  and `lym loom parent-review ask` (new) and `lym loom parent-review await` to a PR-Review session: `internal/shellguard/rows.go` holds no row against them.

## Auto-mode assumptions

None.

## Open risks

None.

## Acceptance criteria

- D1: `beam.yaml`'s template carries `autonomy: false`; the key is classified `CategoryGuard`; a task's `run_settings` of `{"beam":{"autonomy":true}}` is refused by the local-layer check.
- D1: a worktree layer of `false` over a default layer of `true` resolves `false` for that worktree.
- D1: Finalize's config-change notice for a landing whose worktree file changed `autonomy` does not name `autonomy`.
- D2: a run's reader answers `true` when its parent worktree's `beam.yaml` resolves `autonomy: true`, `false` when it resolves `false`, and `false` with a warning when the stack fails to load.
- D2: an orch's reader answers from its own worktree's stack, not its parent's.
- D2: a halt notice of a run carries the run's autonomy read when the notice is built.
- Review: D2 — `role-autonomy.md` has the orch handle a halt under the value its notice carries:
  with the run's parent worktree `false` and its orch down, the prime orch under `true` relays the halt to the operator;
  with it `true`, an ancestor orch under `false` settles it.
- D3: with the parent worktree autonomous and layers holding `pr_review_ask_parent: false`, `judge_ask_parent: false`, `judge_interactive: true`, `discussion_interactive: true` and `raddle_auto_approve: false`,
  PR-Review asks the parent, Drift-Verify runs its parent-agreement gate, the judge and the Discussion lead spawn non-interactive with autonomous mode text, and a raddle proposal files with no approval.
- D3: a profile's raddle block holding `raddle_auto_approve: false` reads `true` under autonomy.
- D3: with autonomy off, each of those keys keeps its own layered value; a raddle run already past its first Raddle-Write call keeps its recorded mode.
- D3: under autonomy, a judge ask that ends timeout past `judge_parent_timeout_min`, not-sent or no-parent stops the run at Drift-Verify with a halt naming `lym loom resume`,
  and the resume sends a new ask over the same rulings; without autonomy the rulings stand in each case.
- D4: under autonomy the PR-Review session spec has `Interactive` false; without autonomy it has `Interactive` true.
- D4: under autonomy each `[pr-review request]` line names `lym loom parent-review answer <slug> --ask <n>` as its answer.
- D4: `lym loom parent-review ask` records a PR-Review ask in the run's ask record, with its tag and the head it read;
  it is refused outside a PR-Review session whose record holds autonomy `true`, and while the newest PR-Review ask stands unanswered at the current head;
  after that ask reaches its deadline unanswered, the answer is refused naming the session's next request as its way forward, and a new `parent-review ask` is recorded.
- D4: under autonomy, with an earlier Code-Review or judge ask in the run, `lym loom parent-review answer <slug> --ask <n> --satisfied` for the newest PR-Review ask is recorded,
  and `lym loom pr-continue` then ends the review.
- D4: under autonomy `lym loom pr-continue` refuses with no satisfied answer, and refuses when the newest satisfied ask was sent at an older head; it succeeds after a satisfied answer at the current head.
- D4: under autonomy, a commit after a passing `lym loom pr-continue` leaves the review open, with the continue flag cleared and the session mailed to ask again, until a new ask at the new head is answered satisfied.
- D4: under autonomy, a pull request closed on GitHub while PR-Review waits halts PR-Review and `lym loom pr-continue` is refused on it;
  one closed after the review ended halts Finalize, which reopens neither.
- D4: under autonomy, a GitHub approval of the gated head ends the review with no parent answer, as today.
- D4: under autonomy, with a mailed head standing and no satisfied answer, `lym loom pr-continue` clears the mailed head and leaves the review open, and the row gates that head again.
- D4: a toggle of autonomy while a PR-Review session runs changes neither the session's interactivity, after a pause or a judging stop resumes the session, nor `pr-continue`'s check, which read the value recorded once for the review.
- D4: under autonomy, with the newest ask unanswered at the current head, `lym loom pr-continue`'s refusal names waiting for the parent's answer, and the gate-failure hint names no operator.
- D4: `lym loom goto` onto PR-Review clears the record's autonomy and session fields, so the session the re-entry starts after autonomy turned off is new and interactive.
- D4: after autonomy turns off while a review waits, the operator's unnamed shell answers the newest PR-Review ask with `lym loom parent-review answer`.
- D4: without autonomy `lym loom pr-continue` succeeds with no parent answer, as today.
- Review: D4 — the PR-Review stencil, under autonomy, asks the parent after each fix with `lym loom parent-review ask`, waits with `lym loom parent-review await`,
  and runs `lym loom pr-continue` only on its satisfied answer.
- Review: D4 — `orch-role.md`'s "Reviewing a pull request" names `role-autonomy.md` as its exception, and `role-autonomy.md` gives the `lym loom parent-review answer` answer.
- Review: D5 — `role-autonomy.md` lists exactly the six severe-halt classes, the `[operator] severe halt <slug>:` opening,
  and class 5's count of earlier halts at the row from the run's status history, each halt entry, a Stuck with no `routed_to`, counting as one halt plus its `repeats`.
- D6: with the beam's orch autonomous, Beam-Await posts one `[beam drained] <slug>` line on the first wake after its last open member lands,
  and none on later wakes, after the row is re-entered or after a dobby-down notice is sent and cleared.
- D6: a member that opens and lands again after a drain causes one further line;
  a beam with no commits beyond its parent, a standing ready record, an open draft member, or autonomy off causes none.
- Review: D7 — `role-autonomy.md` gives the four steps starting at the first not done, step 1 only on a conflict refusal with a beam only behind its parent going to step 2,
  the `[operator] merge ready <slug>:` notice and `lym beam ready` waiting for the operator's word.
- D8: with the presence slack and the orch's worktree autonomous, the reply hook forwards a reply opening `[operator]` and drops one that does not; with autonomy off it forwards both.
- Review: D8 — `role-autonomy.md` has the orch post each severe-halt relay and merge-ready notice to `<shortname>:SLACK` with `lym creel post`, one relay per turn opening its own reply;
  `role-slack.md` has it open every reply meant for the operator with `[operator]` while the presence is slack.
- D9: the start, resume and reload pointers and the pending resume after a handoff carry the orch's autonomy line, on and off, under a slack and a terminal presence.
- D9: after a value change made outside `lym config beam --set`, such as a hand-edited worktree file, the next halt notice carries the new value.
- Review: D9 — `orch-role.md` points to `role-autonomy.md`;
  its operator-word rules, its go-ahead working rule and its "Reviewing a pull request" section, `role-slack.md`'s forwarding rule,
  which also gains D8's marker rule,
  and the `lym beam pr` and `lym beam describe` help in `internal/beamcli/cli.go` name it as their exception.
- Review: D10 — `internal/heddleengine/glossary.md` carries the two lines.

## Notes for the plan writer

- The drift and PR-Review consumers reach D2's run reader through their existing seams, such as `ParentReviewWanted` and the judge's `JudgeAskParent` read in `internal/driftengine/verify.go`.
