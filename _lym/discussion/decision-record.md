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
- Telling the orch its worktree's autonomy at every launch and on every change.
- Glossary lines for the two new terms.

Out:

- Turning member pull requests off (D4).
- Merging the beam's pull request outside Finalize (D7).
- Releasing draft tasks, making tasks or promoting notes: these stay the operator's under autonomy.
- A sampler run's Sampler-Session: the operator picks that recipe to sit in it, so it still waits on the operator.
- Turning autonomy on in the Models hub on OSL-1046 or in loomyard: a config write the operator orders after landing, since a run writes no hub config.

## Decisions

### D1 — `autonomy` is a `beam.yaml` key, a Guard key

**Decision:** `beam.yaml` (`internal/dobbyengine/config.go`) gains `autonomy`, a boolean, default `false`, classified `CategoryGuard`.
It is set with `lym config beam --set autonomy=<true|false>` in a worktree, for that worktree,
or with `--layer default` from the prime, for every worktree of the hub.
A worktree layer overrides the default layer, so a beam can opt out of a hub-wide `true`.
A worktree's file is inherited by a worktree forked from it, as every per-worktree file is,
so a beam or nested beam made under an autonomous worktree starts autonomous until its own file is set `false`.
Finalize's config-change notice (`internal/landingshed/finalize.go`) leaves `autonomy` out,
so a landing never asks an orch to re-apply a child's autonomy on its parent.

**Rationale:** `beam.yaml` is the existing per-worktree module whose `pull_request` a member's landing already reads from its parent worktree;
projects carry no `run_settings`.
Autonomy lets the parent's answer end PR-Review in the operator's place, which hollows an approval gate, so PATTERN-config-layers makes it a Guard key.
As a Guard key the local layer refuses it, so a task's `run_settings` can never make a run autonomous: only a tracked worktree or default file can.
A worktree's autonomy is its own, so it never travels upward on a landing.

### D2 — Whose value applies: a run reads its parent worktree's, an orch its own worktree's

**Decision:** two readers answer autonomy, both through `dobbyengine.LoadConfig`, both at call time, never from a value captured at wiring:

- A run's autonomy is its parent worktree's resolved value, the worktree whose orch its notices reach.
  A root task's parent worktree is the prime; a beam's own pawl run's parent worktree is the worktree the beam was made from.
- An orch's autonomy, and that of the steps it takes for its own worktree (D6, D7, D8, D9), is its own worktree's resolved value.

So an orch applies its own worktree's value to every halt that reaches it, a beam's own Finalize halt included.
A stack that fails to load reads as `false` and is logged as a warning naming the worktree.

**Rationale:** a run's notices reach its parent worktree's nearest live orch (`orchcli.ParentStrand`), so the run and that orch agree on one value.
Reading `false` on a failure falls back to the operator being asked, never to a skipped operator;
dobby loads `beam.yaml` strictly, so a broken file already stops the worktree's dobby, which the orch hears of.

### D3 — Autonomy forces the parent asks on and every operator-seat session off

**Decision:** under a run's autonomy (D2) the run reads these keys as forced, whatever its own layers say:

- `pr_review_ask_parent` (loom) reads `true`.
- `judge_ask_parent` (drift) reads `true`.
- `judge_interactive` (drift) reads `false`.
- `discussion_interactive` (loom) reads `false`.
- `raddle_auto_approve` (loom) reads `true`.

The forcing is applied to the module's resolved config, before any reader uses a field, so a key read in several places takes one value.
No layer file is rewritten, so turning autonomy off restores each key's own value at its next load.
`code_review_ask_parent` and `discussion_review_ask_parent` keep their own values:
this reads the entry's "sets the existing ask keys" as the keys that move an operator wait to the parent.

**Rationale:** the judge ask lets the orch rewrite a drift ruling before a does-not-match stops the run, the largest share of halts,
and the PR-Review ask is how the parent reviews a member pull request (D4).
An interactive Discussion or judge and an unapproved raddle proposal each wait for an operator in a pane, which autonomy rules out;
a raddle filing makes draft tasks, so their release stays the operator's.
The code and discussion review asks add a veto per round but remove no operator wait.

### D4 — Under autonomy, PR-Review ends only on the parent's recorded satisfied answer

**Decision:** whether a member task opens a pull request stays `landingshed.ResolvePullRequest`'s decision, unchanged;
an operator who wants no member pull requests sets the beam's `pull_request` off.
Under a run's autonomy:

- The PR-Review session (`internal/landingshed/prsession.go`) starts with `Interactive` false, and its prompt renders the run's autonomy.
- Every review ask to the parent, lym's first and each re-ask after the session's fixes, goes through the parent-review seat with an ask number,
  as code-review asks do, answered with `lym loom parent-review answer <slug> --ask <n>` and `--satisfied` or `--findings-file`.
- `lym loom pr-continue` refuses unless the newest ask was sent at the pull request's current head and is answered satisfied;
  the refusal names the way forward, asking the parent again.
- On that answer the session runs `lym loom pr-continue`, which ends the review.

The orch still never approves, merges, closes or comments on the pull request on GitHub.
Without autonomy, PR-Review and `pr-continue` behave as today.

**Bound:** only a run whose parent worktree resolves autonomy `true` lets the parent's answer stand in for the operator's,
and only a recorded satisfied answer at the current head ends the review.
PR-Review's own gate on the head and Finalize's merge gate run unchanged;
a GitHub rule that needs a human approval still refuses the merge, which is a D5 class 4 severe halt.
A waiting session's held turns raise hold notices to the parent, as an autonomous session's do today, so an unanswered ask is raised again.

**Rationale:** keeping the pull request keeps the member trail the beam's description lists and keeps PATTERN-landing's resolver the sole decider.
A recorded answer makes the parent's ending mechanical, since D1's Guard rests on it.

### D5 — The orch settles every halt under autonomy; a closed list of severe halts goes to the operator

**Decision:** halt notices keep their routing.
Under its autonomy (D2) the orch settles every halt and run-level call itself, as the role already has it,
and also settles a choice between designs the board entry leaves open, recording it with `lym loom decision add <slug> --by parent`.
It resumes a halted run on its own judgment, never waiting for the operator's word.
A drift does-not-match stop is a run-level call:
a decision and a goto that send the run back to fix the code, or `lym drift amend` when the expectation was wrong.

A severe halt is one of these classes, and no other:

1. An owner halt (`owner check:`) or any takeover of a run onto this machine.
2. A scope change: work the task's board entry rules out or lacks, or a new task, since only the operator makes one.
3. Giving a run up with `lym batten cancel`.
4. A step only the operator can take outside lym: on GitHub (a closed pull request to reopen, a human approval a branch rule requires) or on the machine (credentials, disk).
5. A repeating stop: a halt at a row of a run whose decision record already holds two `--by parent` entries the orch made for halts at that row.

The orch counts class 5 from the run's decision record, which outlives its context cycles, and names the row in each such entry's title.
It files the bug a repeating stop shows before relaying it.
lym does not classify severity: the orch classifies each halt against this list.
Its relay opens `[operator] severe halt <slug>:`, followed by the halt, what the orch tried and the one action it asks for.

**Bound:** the list is closed; a halt outside it is never relayed as severe.
Class 5 bounds the orch's own settlements at two per row of a run.

**Rationale:** the role already settles most halts; the operator waits came from design choices, the resume word and PR endings.
A closed list keeps the operator's two cases checkable, and a fixed opening makes a relay recognisable in the pane and on Slack (D8).

### D6 — Beam-Await posts one `[beam drained]` line when an autonomous beam's work is done

**Decision:** Beam-Await (`internal/pawlengine/beamawait.go`), while the beam's orch's autonomy (D2) is `true`,
posts one `[beam drained] <slug>` line to the beam's orch through its existing `Notify` seam once all of these hold:
the beam holds commits beyond its parent, no member task of the beam is open, drafts included,
no child pair of the beam exists, and no ready record stands.
The posted drain is kept as a marker in the beam run's records, so a re-entry of the row posts no second line;
a member that opens again removes the marker, and the next drain posts again.
The check runs on the row's existing wakes.
With autonomy `false` it posts nothing.

**Bound:** the line only informs; it opens, merges or readies nothing.

**Rationale:** Beam-Await already wakes and already reaches the beam's orch, so it sees a drained beam without a new poller.
Requiring commits beyond the parent keeps a new, empty beam from draining.

### D7 — On `[beam drained]`, the orch readies the pull request and notifies the operator

**Decision:** under its autonomy the beam's orch takes each drained line as its go-ahead for the beam's landing steps up to the pull request,
starting at the first step not yet done:

1. When the beam lags its parent, so `lym beam pr open` would refuse, run `lym beam merge-parent <slug>`.
   Its merge task starts released; its landing reopens and then drains the beam, and that next drained line resumes these steps.
2. When no description is recorded, write it with `lym beam describe <slug> --draft`, edit it, and record it with `lym beam describe <slug>`.
   When one is recorded, add any new member to it and record it again.
3. Run `lym beam pr open <slug>`, which opens the pull request ready for review, never as draft, or finds the open one.
4. Read `lym beam pr status <slug>` and notify the operator with `[operator] merge ready <slug>:`, the pull request's URL and any blocker the status names.

The operator's word to land is the go-ahead for `lym beam ready <slug>`, which the orch then runs; Finalize merges as today.

**Bound:** the go-ahead covers only these four steps for the beam the line names; `lym beam ready` still waits for the operator.

**Rationale:** the orch writes the description, and PATTERN-github-auth lets only `lym beam pr` open the beam's pull request, so lym opens it through the orch's verb.
PATTERN-landing keeps the merge in Finalize, so the operator's act is the word to ready the beam.

### D8 — Under autonomy, the Slack reply hook forwards only operator-marked replies

**Decision:** while the orch's autonomy (D2) is `true`, the Slack reply hook (`internal/slackcli/hook.go`) forwards a turn's reply only when it opens with `[operator]`.
Under autonomy the orch opens every reply meant for the operator with it:
D5's severe-halt relay, D7's merge-ready notice and its answers to the operator's own words, typed or from `<shortname>:SLACK`.
With autonomy `false` the hook forwards every reply, as today.

**Bound:** the filter drops only replies without the marker, and only while the presence is slack; the pane shows every reply.

**Rationale:** the orch's turns on settled halts would otherwise reach the operator on Slack on every notice, against the goal's two cases.

### D9 — The orch is told its autonomy at every launch and on every change

**Decision:** every orch pointer lym sends at a launch or context cycle carries one line naming the orch's autonomy, read then,
on the same path that appends the presence line today (`internal/orchengine/prompt.go`).
A write of `autonomy` through `lym config beam` mails each live orch of the hub whose resolved value it changed, naming the new value.
The orch role (`contracts/orchrole/orch-role.md`) points to a new on-demand file, `contracts/orchrole/role-autonomy.md` (new), to read while its autonomy is on.
The file holds D5's handling and severe-halt list, D4's PR-Review answers, D7's steps and D8's marker.
The rules in `orch-role.md` and `contracts/orchrole/role-slack.md` that wait for the operator's word or forward every reply name the autonomy file as their exception.

**Rationale:** the role is rendered from embedded bytes for every orch, so a per-worktree mode reaches it as a launch fact and an on-demand file, as beams and landing halts do.
The change mail keeps the orch's reading in step with the call-time reads of D2, D3, D4 and D8.

### D10 — Two glossary lines

**Decision:** `internal/heddleengine/glossary.md` gains:

- `- **autonomy**: a worktree's mode in which its orch settles every halt of the runs under it and reaches the operator only for a severe halt or a merge-ready beam. Never: autopilot, full auto.`
- `- **severe halt**: a halt in one of autonomy's closed classes, which the orch relays to the operator. Never: critical halt, blocker.`

**Rationale:** the design adds both terms, and records and notices use them.

## Constraints

- PATTERN-config-layers: `autonomy` is classified in `beam.yaml`'s `ConfigSpec` in the same change, as Guard; no reinterpretation is needed, since the key is new.
- PATTERN-landing: `landingshed.ResolvePullRequest` alone decides a member's pull request, and Finalize alone merges, under the parent lock.
- PATTERN-github-auth: the beam's pull request is opened only by `lym beam pr`, which never merges; the PR-Review session runs no `gh` of its own.
- PATTERN-daemon-wakeups: D6 rides Beam-Await's existing wakes, which back off to one minute.
- PATTERN-agent-addressing: every new line goes through creel, through the seams that post today.
- PATTERN-refusal-way-forward: D4's `pr-continue` refusal names its way forward, and a test reaches it.
- PATTERN-stencil-ownership: `role-autonomy.md` and the pointer line render through the orch role's existing path, `orchengine.RenderRoleFile`.
- The resume, decision, circling, drift-amend and beam verbs D5 and D7 name are open to an orch session: `internal/shellguard/rows.go` holds no row against them.

## Auto-mode assumptions

- Draft members keep a beam open for D6, since releasing them is the operator's.

## Open risks

None.

## Acceptance criteria

- D1: `beam.yaml`'s template carries `autonomy: false`; the key is classified `CategoryGuard`; a task's `run_settings` of `{"beam":{"autonomy":true}}` is refused by the local-layer check.
- D1: a worktree layer of `false` over a default layer of `true` resolves `false` for that worktree.
- D1: Finalize's config-change notice for a landing whose worktree file changed `autonomy` does not name `autonomy`.
- D2: a run's reader answers `true` when its parent worktree's `beam.yaml` resolves `autonomy: true`, `false` when it resolves `false`, and `false` with a warning when the stack fails to load.
- D2: an orch's reader answers from its own worktree's stack, not its parent's.
- D3: with the parent worktree autonomous and layers holding `pr_review_ask_parent: false`, `judge_ask_parent: false`, `judge_interactive: true`, `discussion_interactive: true` and `raddle_auto_approve: false`,
  PR-Review asks the parent, Drift-Verify runs its parent-agreement gate, the judge and the Discussion lead spawn non-interactive with autonomous mode text, and a raddle proposal files with no approval.
- D3: with autonomy off, each of those keys keeps its own layered value.
- D4: under autonomy the PR-Review session spec has `Interactive` false; without autonomy it has `Interactive` true.
- D4: under autonomy `lym loom pr-continue` refuses with no satisfied answer, and refuses when the newest satisfied ask was sent at an older head; it succeeds after a satisfied answer at the current head.
- D4: without autonomy `lym loom pr-continue` succeeds with no parent answer, as today.
- Review: D4 — the PR-Review stencil, under autonomy, asks the parent after each fix and runs `lym loom pr-continue` only on its satisfied answer.
- Review: D5 — `role-autonomy.md` lists exactly the five severe-halt classes, the `[operator] severe halt <slug>:` opening, and class 5's count from the decision record.
- D6: with the beam's orch autonomous, Beam-Await posts one `[beam drained] <slug>` line on the first wake after its last open member lands, and none on later wakes or after the row is re-entered.
- D6: a member that opens and lands again after a drain causes one further line;
  a beam with no commits beyond its parent, a standing ready record, an open draft member, or autonomy off causes none.
- Review: D7 — `role-autonomy.md` gives the four steps starting at the first not done, the `[operator] merge ready <slug>:` notice and `lym beam ready` waiting for the operator's word.
- D8: with the presence slack and the orch's worktree autonomous, the reply hook forwards a reply opening `[operator]` and drops one that does not; with autonomy off it forwards both.
- D9: every orch pointer carries the orch's autonomy line, on and off; a `lym config beam --set autonomy=…` that changes a worktree's value mails that worktree's live orch, and one that changes nothing mails none.
- Review: D9 — `orch-role.md` points to `role-autonomy.md`, and it and `role-slack.md` name it as the exception to their operator-word and reply-forwarding rules.
- Review: D10 — `internal/heddleengine/glossary.md` carries the two lines.

## Notes for the plan writer

- The drift and PR-Review consumers reach D2's run reader through their existing seams, such as `ParentReviewWanted` and the judge's `JudgeAskParent` read in `internal/driftengine/verify.go`.
