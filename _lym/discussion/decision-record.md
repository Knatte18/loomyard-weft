# Decision record — parent-reviews-decision-record

## Goal

Let a hub or a task add an extra review of the Discussion decision record before Plan starts, either by the run's parent orch or by a named agent, or by both.
Every such review is off by default and is switched on per hub in `loom.yaml` or per task in its `run_settings`.
The parent's load stays small: the review rides in the perch's existing round, a fork of the orch can do it, and a deadline lets the round go on without it.

## Scope

In:

- The parent seat: a Discussion-Review round that asks the parent orch to review the record, waits for its answer up to a deadline, and folds its findings into the round.
- Two new `loom.yaml` keys for the parent seat, both open to a task's `run_settings`.
- An answer verb the parent runs and a capped wait verb the reviewer chair runs.
- The orch's instructions for the new request, in `contracts/orchrole/orch-role.md`.
- Confirming that the named-agent fan of issue #113 is the existing Discussion-Review assistant reviewer, settable per task.

Out:

- The Plan and Code perches: the board entry says the option "could later" apply there; nothing here builds it.
- `pr_review_ask_parent` and the PR-Review row: unchanged.
- A separate review step or row between Discussion-Review and Plan.
- Discussion-Write and its own lens forks.

## Decisions

### D1 — The named-agent fan is the existing assistant reviewer

The agent fan issue #113 asks for already exists: `discussion_assist_reviewer_model` names the assistant's model-spec, and `discussion_assist_reviewer_fan` its lens fan, each per round.
Both are in `internal/loomengine/template.yaml`, default empty, and open to a task's `run_settings`.
`fable[high]` is a valid model-spec for it: `contracts/specs/llm-model-spec.md` maps `fable` to the Fable model family.
The assistant reviews in the same round as the chair's lens forks, and its findings join the round as a lens's do.
This task adds no agent-fan key and no code for it; the new orch-role section (D6) names these keys as the way to add an agent review.

### D2 — The parent seat is a seat of the Discussion-Review round, not a step before Plan

A new `loom.yaml` key `discussion_review_ask_parent`, default `false`, turns the parent seat on.
It is category `none` in `loomengine`'s `ConfigSpec`, so a task's `run_settings` overrides the hub's value either way.
Each Discussion-Burler-Review round reads the key; with it off, the round asks nothing, whatever an earlier round did.
With it on, an asking round asks the parent orch to review the decision record at the same time as the chair's lens forks and any assistant review it.
The parent's findings join that round's review, Discussion-Burler-Fix fixes them in the same round, and the Discussion-Bouncer judges the round as it does today.
No row, pause, extra round or parent decision is added.

Which rounds ask, counted within one Bouncer generation (a fresh `lym loom start`, a goto back to Discussion-Write and an amendment each start a new generation, while round numbers carry on):

- The first round of a generation always asks, so every rewritten or amended record goes to the parent.
- A later round of the same generation asks only when the parent's answer in that generation's round before held findings, so the parent can confirm their fixes.
- Otherwise the round asks nothing.

Bound: the parent's answer never gates the round by itself; approval stays the Bouncer's.
This departs from issue #87's "Plan starts only on approval": when the Bouncer approves after Burler-Fix fixed the parent's findings, no later round runs and the parent is not asked to confirm; the fixed record goes to Plan.
A `discussion_max_bounces` of `0` runs no round, so the parent is never asked;
`discussion_auto_accept`, a hard cap or `lym loom circling accept` can end the perch with parent findings open, as with any other finding.
Rationale: a seat in the round costs no extra round, reuses Burler-Fix and the Bouncer, and keeps the parent's load to one read per generation unless it found something.

### D3 — Go asks the parent once per round, through creel

When a round asks (D2), the Discussion-Burler-Review row sends the parent orch one creel line before the review table starts, through `orchcli.NotifyParent`, the path PR-Review's parent ask uses.
The line opens `[discussion-review request]` and names the slug, the round, the decision record's path in the task worktree, the deadline as a wall-clock time, and the answer verb (D4).
The ask, its deadline and its state live in a never-tracked ask record under the run's `.lym` directory, separate from the Bouncer's per-round parent notice.
The ask record is keyed by the Bouncer generation and the round number, never by the review attempt token.

- A restarted or resumed review row of the same round finds the ask record and sends no second ask; it keeps the first ask's deadline.
- An ask that cannot be posted, because no live orch resolves or the post fails, sets the state `no-parent` at once and logs it; the round goes on without the parent.
- An unreadable `discussion_review_ask_parent` is logged and treated as `false` for that round.

Bound: the ask carries only the slug, round, path, deadline and verb; the parent reads the record itself.

### D4 — The parent answers with `lym loom parent-review answer`

A new verb, `lym loom parent-review answer <slug>`, takes exactly one of `--satisfied` or `--findings-file <path>`.
It is a `loom` cobra subcommand in `internal/loomcli`, run from the caller's own worktree as `lym loom decision add <slug>` is.
It writes the parent's report, never tracked, beside the round's ask record and sets the ask `done`.

- `--satisfied` writes a report with no findings.
- `--findings-file` reads a markdown file of findings, each naming what to change; the verb copies it into the report and refuses an empty file.
- Caller guard: the same as `lym loom decision add --by parent`, whose guard in `internal/loomcli/decision.go` admits the run's parent orch by strand name and the operator's own unnamed shell.
  Any other caller is refused, naming the way forward: the run's parent orch answers.
- The verb refuses when the slug has no open ask: no ask this round, or one already `done`, `timeout` or `no-parent`.
  The refusal names the way forward: a call the parent still wants made goes in with `lym loom decision add <slug> --by parent`.
- Once the review table ends, `burlerengine` copies a `done` report beside the review file, as `keepAssistReports` does the assistant's, so it is committed with the round.
  The copy joins the outputs the review row archives on a failed exit, beside the assistant's copy.

Bound: the verb writes only the parent report and the ask's state for the slug's current round; it edits no tracked file and never the decision record.

### D5 — The deadline: `discussion_review_parent_timeout_min`, and a capped wait the chair runs

A second new key, `discussion_review_parent_timeout_min`, default `30`, category `none`, sets the minutes from the ask to its deadline.
It must be a positive integer; `loomengine`'s config validation refuses `0` or a negative value, naming the key.
The deadline is capped so it falls before the chair's own `review_timeout_min` runs out, as `assist_reviewer_timeout_min` is capped, so the chair always outlives the wait.

The chair waits for the parent with a new verb, `lym loom parent-review await <slug>`, run in the foreground before it consolidates.
A new verb, not `lym shuttle await` on a marker, because something must set `timeout` at the deadline, and the parent is no shuttle run whose end writes a marker.

- It answers `ready: true` once the ask is `done` or `no-parent`.
- At or past the deadline with the ask still open, it sets the ask `timeout` and answers `ready: true`.
- Otherwise it waits, waking at most once a second, up to the keep-warm cap `shuttleengines.KeepWarm` gives the chair's strand, then answers `ready: false`, and the chair runs it again.

The ask's state moves only out of `open`, under one per-run lock both verbs take, so an answer and a timeout never both land: the second is refused (answer) or reads the first (await).

On `timeout` or `no-parent` the chair consolidates without the parent and writes one line in the review saying the parent review did not arrive and why; that line and the ask record are the record that the round went on without it.

Bound: the capped deadline is the longest the parent can hold a round.

### D6 — The orch's instructions: a fork may do the whole review and answer it

`contracts/orchrole/orch-role.md` gets a section for mail from `<shortname>:LOOM` opening `[discussion-review request]`.
It tells the orch to hand the review to a one-shot fork: the fork keeps the orch's knowledge of the run, reads the decision record and the run's board entry and settled calls, and answers itself, so the review stays out of the orch's main thread.
Such a fork may run two writing verbs, both for the slug the request names: `lym loom decision add <slug> --by parent` for a call it wants made rather than a fix, before its answer, and `lym loom parent-review answer`.
The section states this as an exception to the forks-only-read rule.
The section names D1's assistant keys as the way to add an agent review, and both D2 and D5 keys with their `run_settings` override.

Fact the fork rests on: `lym loom` verbs carry no fork check, and the caller guard reads the caller's strand name, which an in-session fork shares with the orch, so the fork passes D4's guard.
Bound: the fork's limit to those two verbs and that slug is instruction text; no code enforces it beyond D4's caller guard.
Rationale: the operator asked that a busy orch can keep the review off its main thread.

### D7 — The chair keeps every parent finding

The review step gets a parent consolidation part, filled only in a round that asked the parent, beside the assistant's part.
It tells the chair to wait with D5's verb, then read a `done` parent report and keep every finding in it, tagged `origin: parent`, with the parent's severity, `BLOCKING` when the parent gives none.
A parent finding keeps the class the parent gives when it is `design`, `scope` or `consistency`; a `decision` class or none becomes `design`, so no parent finding escalates back to the parent.
The chair may merge a parent finding with a duplicate of its own but never moves one to `## Rejected`.
The chair's verdict reflects open parent findings like any other kept finding.

Rationale: the parent is the operator's delegate, so a reviewer agent does not overrule it.

## Constraints

- `PATTERN-config-layers`: both new keys go through `configengine`, with defaults in `internal/loomengine/template.yaml`; no meaning of an existing key changes, so no `hubreconcile` reinterpretation ships.
- `PATTERN-agent-addressing`: the ask goes through creel, via `orchcli.NotifyParent`; the parent is resolved by the existing parent routing.
- `PATTERN-refusal-way-forward`: D4's refusals name their way forward, and a test reaches each refusal.
- `PATTERN-daemon-wakeups`: D5's wait wakes at most once a second and takes its cap from the keep-warm resolver.
- `PATTERN-lym-files`: the ask record and the parent report live under `.lym`, named through `lymdirs`, and D5's lock path comes from `lock`; the copy beside the review is tracked under `_lym` like the assistant's copy.
- `PATTERN-tolerant-records`: the ask record is Go-written JSON written through `internal/state` and refuses no unknown key.
- `PATTERN-cli-cobra`: `lym loom parent-review` is a cobra subtree of the `loom` module.
- `PATTERN-stencil-ownership`: the parent consolidation part is a stencil read at call time through `stencilstore`.
- `PATTERN-run-records`: the parent report is run state, not a friction note, usage record or project record; the decision record is still written only by its writers.

## Auto-mode assumptions

- The defaults and picks of D2, D5 and D7 are auto-picks; the support log's ledger gives each one's rationale.

## Open risks

- None.

## Acceptance criteria

- D1: a task whose `run_settings` sets `discussion_assist_reviewer_model: fable[high]` over a hub that sets none runs its Discussion-Review round with an assistant seat on that model; existing coverage may meet it.
- D2: with `discussion_review_ask_parent` off, a Discussion-Burler-Review round sends no `[discussion-review request]`; with it on in the hub, or on in a task's `run_settings` over a hub that leaves it off, the generation's first round sends one.
- D2: a task's `run_settings` setting `discussion_review_ask_parent: false` over a hub that sets it `true` sends no ask.
- D2: a later round of the same generation asks again when the round before's parent answer held findings, and asks nothing when it was satisfied, timed out or `no-parent`.
- D2: the first round of a new generation asks, whatever the previous generation's last parent answer was.
- D3: the ask line names the slug, the round, the decision record's path, the deadline and the answer verb.
- D3: a review row restarted in the same round, with a new attempt token, sends no second ask and keeps the first deadline.
- D3: with no parent orch to reach, the ask is set `no-parent`, the round goes on, and the review records that the parent review did not arrive.
- D4: `--satisfied` sets the ask `done` with a report that holds no findings; `--findings-file` sets it `done` with the file's findings; an empty findings file is refused.
- D4: the answer verb is refused for a slug with no open ask and for an ask already `done` or `timeout`, each refusal naming `lym loom decision add`; it is refused for a named caller that is not the run's parent orch.
- D4: a `done` parent report is copied beside the review file once the review table ends, and a failed review exit archives that copy.
- D5: the await verb answers ready once the ask is answered; at the deadline with no answer it sets the ask `timeout` and answers ready; before both it answers not ready once its cap runs out.
- D5: with `review_timeout_min` shorter than `discussion_review_parent_timeout_min`, the deadline falls before the chair's timeout.
- D5: an answer racing the deadline leaves exactly one of `done` or `timeout`, and the losing answer is refused.
- D5: `discussion_review_parent_timeout_min: 0` is refused by config validation naming the key.
- D7: in a round whose parent report holds findings, the review file carries each of them with `origin: parent`, a parent finding of class `decision` carries class `design`, and none sits under `## Rejected`.
- Review: D6 — `contracts/orchrole/orch-role.md` has a section for `[discussion-review request]` that hands the review to a one-shot fork, lets that fork run `lym loom decision add --by parent` and `lym loom parent-review answer` for the named slug only, and names the D1 assistant keys and the D2 and D5 keys.
- Review: D7 — the parent consolidation stencil tells the chair to wait with the await verb, keep every parent finding with `origin: parent`, map `decision` to `design`, and never reject one.
- Review: the `internal/loomengine/template.yaml` comments for both new keys say they are settable per run through `run_settings`.

## Notes for the plan writer

- The assistant seat in `internal/burlerengine/assist.go` is the closest model for the copy beside the review and the optional consolidation marker in `burler-step-review.md`, which a round with both an assistant and the parent fills with both parts; the parent is not a shuttle seat, so it is not added to the seat table, and its ask record is keyed unlike `assistPaths`.
- `contracts/orchrole/orch-role.md` is also changed by the peer run `beam-orch-root-project`, in a different section.
