## Goal

Stop waking the hub orch for every parent review and every PR-Gate review of a loom run.
A cold-start alignment agent, spawned by Go at those two points, reads the board, the records of the other runs in flight and what landed on the parent since the run's base, and checks the run against them.
It settles the review itself (approve, or reject on a scope deviation) when the run overlaps no run in flight, and escalates to the run's parent, today `ly:orch`, only on a substantive overlap or a named question it cannot settle.

## Scope

In:

- The alignment agent at loom's parent review (inside Discussion-Write's `parent-review` gate) and at loom's PR-Gate (a new `PR-Align` row before `PR-Gate`).
- A Go-applied verdict: the agent writes `approve`, `reject` or `escalate` with a report; Go records it through the same stores the existing verbs write.
- A read-only verb, `lyx loom peers`, that lets an agent in one pair read the other in-flight runs' records and the parent's landed commits without reaching into another worktree.
- `loom.yaml` keys for the agent's model, timeout, an off switch and whether an agent approval at PR-Gate lands on its own, classified for the config layers.
- The orch role stencil: parent review and PR-Gate reach the orch only as escalations carrying the alignment report; the orch records operator decisions where the agent reads them (D8).
- `CLAUDE.md`'s production paragraph, which this task changes (D5).
- Docs land per `CLAUDE.md`'s same-commit rule, with `contracts/specs/refusal-spec.md` rows for every new refusal.

Out:

- darn: its PR-Gate stays the orch's review point, and its recipe is unchanged.
- Holding or pausing the overlapping run: on an escalation the orch holds and aligns both runs with its existing verbs.
- `orch-split`'s run driver and conversational orch; the agent escalates to "the run's parent", so that split can re-point it without a change here.
- Running the orch on another model.
- Merge-Review and the Webster and Discussion review segments.

## Decisions

### D1 — One alignment agent, spawned by Go at two points

Decision: A new told-geometry package (working name `internal/alignment`) owns the agent: its launch spec through shuttle, its prompt render through `internal/stencilstore` from one stencil (`loom-template-alignment.md`) with a point marker (`parent-review` or `pr-gate`), the verdict schema and validation, and the attempt records.
The spawn names a loom-owned `agentname` role (`alignment`) and a color segment on the launch spec.
The stencil renders the parent directive and the edit directive like every spawned role; its own limits forbid any message, so a question the agent cannot settle becomes an `escalate` verdict, as the arbiter's stencil does.
`internal/parentreview` keeps its import set: the gate reaches the spawn through a seam closure wired in `internal/loomcli`.
Rationale: one agent and one stencil serve both points, and Go, not an LLM driver, decides when it runs, so no orch or driver turn is spent to start it.

### D2 — The agent reads, judges and writes only its verdict and report

Decision: At parent review the agent reads the brief, the run's discussion, the board (`lyx board list`, `lyx board get`) and `lyx loom peers`.
At PR-Gate it reads the run's decision record, plan and diff, the board and `lyx loom peers`.
It may run read-only git.
It writes exactly two files at paths it is told, every time: the verdict file and a report in the burler review format (one with no findings on an approve), so shuttle's completion check sees a fixed output set.
It runs no `lyx loom review`, `approve`, `reject`, `decision`, `goto` or resume verb, writes no board entry, sends no message and edits nothing else.
Rationale: a Go-applied verdict keeps the approval verbs' callers unchanged and lets Go check that the verdict belongs to the spawn before it lands anything.
Bound: the verdict and report paths carry a per-spawn token that only the agent's prompt names, and Go reads a verdict only from the attempt it spawned and only after that spawn; at PR-Gate it applies one only for the task head and pull request number it recorded at spawn, and at both points it applies an approve only against an unchanged decision snapshot (D5).
This keeps a stale file or a guessed path from being applied; an agent in the same pair that deliberately reads the agent's prompt file could still forge one, and that is left to the writer's and driver's stencil limits.
This residual is no wider than today's, since the approval verbs carry no caller guard and an agent in the pair can already run them; the per-spawn token and the attempt and head checks defend against a stale or guessed file only.

### D3 — What the agent judges and when it escalates

Decision: At parent review it runs today's scope check (the board entry's goal and Done-when, no more and no less, no duplication or contradiction of another entry) plus the overlap check.
At PR-Gate it runs today's PR-Gate fork check (the run against its board goal and the operator's recorded decisions, with the carried-over `Merge-Review` open risks) plus the overlap check.
The overlap check reads every other run in flight (D6's peer set) and the parent's commits since the run's base.
It escalates on a substantive overlap: a shared Go package or code file, a decision that contradicts or duplicates one in a peer's record or board entry, or a change to the same PATTERN entry.
A shared bookkeeping file only (`cmd/lyx/testdata/test-budget.yaml`, a module-table line in `docs/overview.md`, a new PATTERN index line) is named in the report but does not escalate, since Merge-In and Merge-Review settle it.
It also escalates on a question that needs an operator decision the board does not record; the report names that question and the record or entry that leaves it open, and an escalation without a named question is invalid and falls back like a failure.
Otherwise it approves, or rejects on a scope deviation.
An approve at PR-Gate lists in its report the board entries the run supersedes.
Rationale: the orch's value at these points is aligning runs and carrying operator decisions; a run that overlaps nothing gains nothing from an orch wake.

### D4 — Parent review: the gate spawns the agent before any delivery to the orch

Decision: When the `discussion` gate entry passes and the `parent-review` gate opens a round, the gate spawns the alignment agent instead of typing the delivery prompt, independent of the reviewer-liveness hold, and polls its verdict and liveness at the gate's existing re-evaluation cadence.
The round records the spawn (attempt id and strand) under its lock, so a driver restart attaches to the live spawn or reads its verdict, and never spawns a second agent for the round.
- `approve`: Go records the round's verdict through the store's plain record, never the cap-reject supersede path.
  An approve against a decision snapshot (D5) that changed since the spawn is applied as an escalation, since a decision sent to the Discussion-Write session or recorded as a board note per D8 can land while the agent runs.
- `reject`: Go records the reject with the agent's report, and the existing re-prompt and the `attempts: 2` cap apply unchanged.
- `escalate`, no verdict by `alignment_timeout_min`, a dead agent or an invalid verdict: the gate falls back to today's delivery prompt, and the reviewer brief names the alignment report, or says why no verdict came.
  The reviewer-liveness hold and `noReviewer` govern this fallback only.
  On any fallback Go stops a still-live agent through shuttle's stop and closes the attempt, so a verdict written after the fallback is never applied.
- A verdict recorded by the parent's verbs while the agent runs wins: Go stops the agent through shuttle's stop and drops its verdict.
  A `lyx loom review notify` while the agent runs is held until the agent ends, then counts as today.
While the agent runs, the keep-alive clock runs from the round's opened-at, so the writer's cache stays warm as it does after a delivery.
The wait bound (`parent_review_wait_min`) and expiry probe are unchanged and cover the agent and an escalated orch review together.
Rationale: the writer is idle and already held by the gate, so the agent's run costs no extra hold, and every failure ends in today's path.
Bound: the agent can approve a discussion without the orch; it cannot pass one past the reject cap, and a second reject still halts the run `blocked` for the parent.

### D5 — PR-Gate: a `PR-Align` row before `PR-Gate` in loom

Decision: `loom-recipe.yaml` gains a Go producer row `PR-Align` between `Publish` and `PR-Gate`; darn's recipe does not.
It skips, spawning nothing, when `PR-Gate` would pass without a pull request (the parent branch is not in `require_pr_to_base`), when `skip_alignment` is set, or when an approval or rejection record already matches the pull request and head.
Otherwise it reads the pull request number and head through `internal/githubclient`, spawns the agent, waits up to `alignment_timeout_min`, and applies the verdict through the same write path `lyx loom approve` and `lyx loom reject` use, shared rather than copied:
- `approve`: writes the approval record, so `PR-Gate` lands the run with no orch wake.
  When `alignment_pr_approve` is false, an approve is applied as an escalation whose report recommends approval.
- `reject`: writes the rejection record, so `PR-Gate` routes to `PR-Rework`.
  The agent's reject is allowed once per run, counted in a durable attempt record under the run's directory that survives restarts and reroutes; a later reject verdict is applied as an escalation, and an orch reject does not count.
- `escalate`, timeout, dead agent, invalid verdict, a head or pull request number that moved since the spawn, or an `approve` against a decision snapshot that changed since the spawn: writes no approval or rejection record and records the report path or the reason.
  The decision snapshot is the content hash of the run's decision record and of the board notes naming the task (D8), recorded at spawn and compared when the verdict is applied.
  `PR-Gate`'s awaiting `ParentNotice` names it, so the driver's relay carries it to the parent, and batten's fallback awaiting notice names it too.
- An approve whose report names superseded board entries queues one notice to the orch through `orchengine`, naming the entries to amend; an approve with none queues nothing.
`PR-Align` ends `Done`, never `Stuck`, except on an error a producer returns today.
Its wait wakes at most once a second and honours pause and cancellation; a pause, goto or teardown stops the agent through shuttle's stop, and a verdict after it is dropped by the attempt and head check.
On re-entry after a restart it reuses a live spawn or a verdict for the same head and pull request.
`PR-Align` takes no parent lock: an idle release during its wait is possible, and `PR-Gate` retakes the lock and reroutes through `Merge-In` and `PR-Align` if the parent moved.
Rationale: a row of its own makes the wait a visible stage in `lyx loom status`, and every re-entry after `PR-Rework` or a moved-parent reroute passes through it again with the new head.
Bound: an agent approval lands the task on the parent and, through batten's Land row, runs the hub's `deploy` to `prod` with no orch or operator turn.
The agent runs the same check the orch's PR-Gate fork runs today, and code review stays with Webster-Review and Merge-Review as today, where Merge-Review may have auto-accepted a spent budget whose open risks the agent reads per D3.
What an agent approval lets through that an orch approval would not is a conflict with an operator decision the agent cannot read, bounded by D8's decision-recording rule, which puts a held task's decision in its decision record or, where none exists, in a board note naming the task.
A decision recorded while the agent runs moves neither the head nor the pull request, so the decision-snapshot check, beside the head and pull request check, turns that approve into an escalation.
`alignment_pr_approve: false` withdraws the landing.
This changes `CLAUDE.md`'s production paragraph, which names the orch's landing as the one exception; that paragraph is updated in the same commit.
The agent can trigger at most one rework per run on its own.

### D6 — `lyx loom peers`: the in-flight records, read from any pair

Decision: A new read-only verb under `lyx loom`, for agents, run from a task pair or the prime.
Its peer set is every board task whose status is a run status (any `<state> · <producer>` value, as boardengine's run lock recognizes it), whatever the state, the caller's own task excluded.
Its default form prints one envelope: for each peer its slug, title, status, parent branch, the paths it changes against its merge base with its parent, and which of its records exist (decision record, plan); and for the caller's run, the parent's commits since the run's merge base (short SHA and subject).
From the prime, which has no run, it lists every task holding a run status and omits the parent's commits.
`lyx loom peers show <slug> --record decision|plan` prints one peer record's content.
The caller's geometry resolves through `internal/cliwire`; a peer's pair path comes from the hub's pair layout as `internal/fabricengine` defines it, and git reads go through `internal/fabricengine`.
A peer whose pair, branch or records are missing or unreadable is listed with what could not be read, never dropped and never an error.
`show` refuses, naming a way forward, an unknown slug, a slug whose status is not a run status, the caller's own slug, and an unknown record name.
With no board dir (a standalone run) the verb refuses naming that no board exists, and both alignment points fall back to today's path without spawning.
Rationale: the board note's open question; an agent in one pair reads another pair's records without forming another worktree's path.
Bound: read-only; it reads only tasks whose board status is a run status, and for each only its board fields (title, status, parent branch), its decision record, plan and changed-path list, plus the caller's parent's commit list (short SHA and subject).

### D7 — Config keys

Decision: `loom.yaml` gains:
- `alignment` (model-spec, default `opus[high]`, empty takes `opus[high]`);
- `alignment_timeout_min` (default 20, 0 takes 20);
- `skip_alignment` (default false; true sends both points to the orch exactly as before this task);
- `alignment_pr_approve` (default true; false makes an agent approve at PR-Gate an escalation, so the orch gives the yes).
Loom's coherence check refuses a resolved `alignment_timeout_min` at or above the resolved `parent_review_wait_min`, naming both keys.
`alignment_timeout_min` is open to the local layer and `run_settings`, bounded by the coherence check: a shorter timeout only routes a review to the orch.
`alignment`, `alignment_pr_approve` and `skip_alignment` are closed to it, since a weaker model or a re-opened auto-landing hollows the approval gate;
a local `skip_alignment: false` under a hub `true` hands both points back to the agent and re-opens the PR-Gate landing.
Rationale: the defaults follow the arbiter's.
Unlike the arbiter's and the reviewers' model keys, which are open to the local layer, `alignment` picks the model whose approve lands on the parent with no later gate, which meets `PATTERN-config-layers`' Guard category.
`alignment_pr_approve` lets the operator take back the PR-Gate landing alone (the board's open question) while keeping the parent-review automation.

### D8 — The orch stencil

Decision: The orch role stencil's Parent-review section says a parent-review request reaches the orch only when the alignment agent escalated or gave no verdict, and the brief names the alignment report the fork reads first.
Its PR-Gate section says the same for an awaiting PR-Gate, that an agent-approved run lands with no orch turn, and that the orch amends the superseded board entries a queued alignment notice names.
An escalation naming a peer that has its own open escalation is handled together with that peer's.
A new rule under The board: an operator decision made in the orch's chat is recorded in the same turn, split by holder.
A decision on an entry no run holds is written into that entry's body through `lyx board`.
A decision on a task a run holds is recorded with `lyx loom decision add <slug> --by parent`, which appends it to that run's decision record;
when the verb refuses because the run has no decision record (batten's pre-loom rows such as `running · Worktree-Create`, `Preflight` and `Loom-Preflight`, a halt before Discussion-Write, or any darn run), the orch records the decision as a board note of its own through `lyx board upsert` with kind note, naming the held task's slug;
only when the verb refuses because Discussion-Write is running and owns the record does the decision go by message to the Discussion-Write session.
The board's run lock is unchanged.
The After landing section's line for an empty `deploy` key runs `update-plugins.sh` on the landing notice, or at the next wake after any landing, instead of at the next wake after an approval.
Rationale: the agent reads the board for an entry no run holds, and a held entry's run lock refuses a body write, so a running task's decision goes to its decision record, which D2 has the agent read at PR-Gate and D6's `show --record decision` serves to peers.
Where no record exists, a note of its own is the write the run lock allows (its refusal names it), and the agent reads it through `lyx board list` per D2.
A decision outside both is invisible to the agent.

## Constraints

- `PATTERN-told-geometry`: `internal/alignment` is handed every path and imports no `lyxcwd`.
- `PATTERN-completion-signal`: the agent's output set is fixed (verdict and report), whatever the verdict.
- `PATTERN-config-layers`: the new keys are classified as D7 says, and the test that pins every template key covers them.
- `PATTERN-shed-recipe-registry`, `PATTERN-shed-producer-seam`: `PR-Align` registers in `internal/shedrecipe`'s map; `shedengine` gains no import.
- `PATTERN-parent-lock`: `PR-Align` takes no parent lock.
- `PATTERN-github-auth`: `PR-Align`'s pull request read goes through `internal/githubclient`.
- `PATTERN-orch-pane-single-writer`: the superseded-entries notice is queued through `orchengine`.
- `PATTERN-durable-vs-ephemeral-state`: verdicts, reports and attempt records are run records under `_lyx`, beside the arbiter's.
- `PATTERN-hubforge-fixtures`: a `lyx loom peers` test that needs several live pairs uses `hubforge` through `fabriccli.CloneAndWire`.
- The new package, verb, row and stencil follow every other PATTERN entry as written; none is changed.

## Auto-mode assumptions

- An agent approval at PR-Gate lands without an orch yes (D5, Q7), withdrawable through `alignment_pr_approve`.
- The agent may reject on a scope deviation (D3, Q5).
- A bookkeeping-file overlap does not escalate (D3, Q4).
- loom only (Q2).
- Default model `opus[high]` (D7, Q11).

## Open risks

- A wrong agent approval reaches `prod` (D5's bound).
- A peer that has not reached Discussion-Write has only its board entry to compare, so an overlap in an undesigned peer can be missed until that peer's own check.
- The agent reads more per review than the orch's fork did, so spend per review rises while orch wakes fall; no measurement exists yet.
- An escalation late in the `parent_review_wait_min` window leaves the orch less time before the gate passes with a Warn.
- `runs-overview`, in flight, also edits `internal/battenshed`'s notice code and the `battenshed` and `loomcli` doc files; expect a merge there.
- `orch-split` will restructure the orch role stencil D8 edits; D8's decision-recording rule puts a running task's operator decisions in its decision record, which a run driver could read too, but leaves `orch-split`'s open question to that task.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d5-bound-misses-decision-during-pr-align (class design, severity MEDIUM, open in rounds 2)
  - d8-board-decision-vs-run-lock (class design, severity MEDIUM, open in rounds 1, 2)
  - skip-alignment-local-layer-reopens-gate (class design, severity BLOCKING, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - note-records-moved-head (class consistency, severity MEDIUM, open in rounds 2)
  - poll-close-errors-no-outcome (class design, severity MEDIUM, open in rounds 2)
  - pralign-call-branches-untested (class scope, severity MEDIUM, open in rounds 1, 2)
  - session-start-stop-guards-untested (class scope, severity MEDIUM, open in rounds 1, 2)
  - snapshot-read-failure-no-outcome (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- Parent review with no peer and an in-scope discussion: the round records an approve from the agent, and no delivery prompt is typed and no message reaches the orch.
- Parent review with the orch not live: the agent still spawns and its approve passes the gate.
- Parent review with a scope deviation: the round records a reject with the agent's report and the writer is re-prompted; a second reject halts the run `blocked`.
- Parent review with a substantive overlap, or an escalation with a named question: the writer gets today's delivery prompt and the brief names the alignment report.
- Parent review where the agent times out, dies, or writes an invalid verdict or an escalation with no named question: today's delivery happens, the brief says why no verdict came, and a verdict the agent writes after the fallback is not applied.
- Parent review where the parent records a verdict while the agent runs: the parent's verdict stands and the agent is stopped.
- A driver restart during parent-review alignment spawns no second agent and still applies the verdict.
- `skip_alignment: true`, or no board dir: both points behave exactly as before this task and nothing is spawned.
- PR-Gate approve: `PR-Align` writes an approval matching the pull request and head, `PR-Gate` ends `Done`, and no awaiting notice reaches the orch.
- PR-Gate approve with `alignment_pr_approve: false`: no approval or rejection record, and `PR-Gate` awaits naming the report.
- PR-Gate approve naming superseded entries: one orch notice is queued naming them.
- PR-Gate reject: a rejection record routes the run to `PR-Rework`; a second agent reject in the same run, across a restart, is escalated instead.
- PR-Gate escalate: no approval or rejection record, and `PR-Gate`'s awaiting parent notice names the alignment report path.
- A decision added to the run's decision record, or a board note naming the task, while `PR-Align`'s agent runs turns the agent's approve into an escalation; the same holds for a decision recorded during parent-review alignment.
- A head or pull request number that moved between spawn and verdict, or a verdict file present before the spawn: the verdict is not applied.
- A driver restart during `PR-Align` reuses the live spawn or the same-head verdict and spawns no second agent.
- A pause or goto during `PR-Align` stops the agent, and its later verdict is not applied.
- A parent branch outside `require_pr_to_base`: `PR-Align` ends `Done` without spawning.
- `lyx loom peers` lists every other task holding a run status, one in the awaiting or blocked state included, and never the caller's, refuses with no board dir naming that no board exists, reports a missing pair, branch or record on its line, lists the parent's commits since the run's base, prints a peer record with `show`, and refuses each `show` input D6 names with a way forward.
- Loom's coherence check refuses a resolved `alignment_timeout_min` at or above the resolved `parent_review_wait_min`, including when a `run_settings` override sets it.
- A local-layer file or a task's `run_settings` setting `alignment`, `alignment_pr_approve` or `skip_alignment` is refused, while `alignment_timeout_min` is accepted.
- The darn recipe has no `PR-Align` row.

## Notes for the plan writer

- The arbiter (`internal/loomcli/arbiter.go` and its siblings) is the nearest precedent for spawning a cold agent through shuttle, waiting on its verdict file, attaching to a live spawn after a restart, and keying records by attempt.
- The parent-review gate's evaluation order (`noReviewer`, the liveness hold, the notify and prompt switch) is in `internal/parentreview/gate.go`; the delivery prompt and brief stencils are `loom-template-parent-review-delivery.md` and `loom-template-parent-review-brief.md`.
- An awaiting producer's parent text travels in `OutputPointer.ParentNotice`, and batten holds its own notice while the driver relays it (`internal/battenshed/notice.go`).
- New tests may push packages past `cmd/lyx/testdata/test-budget.yaml`.
