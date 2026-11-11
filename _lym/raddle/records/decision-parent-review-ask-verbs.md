## Goal

The parent-review ask record, its deadline keys, and lym loom parent-review answer and await

Takes: the Sketch's decision record; CapAssistTimeout, the decision add --by parent guard, KeepWarm, lock and state
Delivers: the loom.yaml keys discussion_review_ask_parent and discussion_review_parent_timeout_min with validation and template comments; the never-tracked ask record and parent report under .lym, with forward-only state moves under one per-run lock; the capped-deadline function; lym loom parent-review answer and await
Exports: the ask record's read, write and state-move API in burlerengine, the deadline cap in loomengine, lym loom parent-review answer <slug> --ask <n>, lym loom parent-review await <slug>

## Scope

The part changes these packages:

- internal/loomcli (about 450 changed lines)
- internal/burlerengine (about 280 changed lines)
- internal/loomengine (about 110 changed lines)
- internal/lymdirs (about 5 changed lines)

## Decisions

### D4 — The parent answers with `lym loom parent-review answer`

A new verb, `lym loom parent-review answer <slug> --ask <n>`, takes exactly one of `--satisfied` or `--findings-file <path>`; `--ask` is required.
It is a `loom` cobra subcommand in `internal/loomcli`, run from the caller's own worktree as `lym loom decision add <slug>` is.
It writes the parent's report, never tracked, beside the ask record and sets the ask `done`.
The report gives each item an id, `P1`, `P2` and so on, in the file's order.

- `--satisfied` writes a report with no findings.
- `--findings-file` reads a YAML file holding a `findings` list.
  Each item has a required `change`, naming what to change and where,
  and an optional `severity` and `class` in the review format's words: `BLOCKING`, `MEDIUM`, `LOW` or `NIT`, and `design`, `scope`, `decision` or `consistency`.
  The verb writes the items into the report.
  It refuses a file with no item, an item with no `change`, and a severity or class outside those words, each refusal naming the file's shape as the way forward.
- Caller guard: the same as `lym loom decision add --by parent`, whose guard in `internal/loomcli/decision.go` admits the run's parent orch by strand name, resolved at call time, and the operator's own unnamed shell.
  It also admits the orch strand the ask record stores for the ask `--ask` names, so an orch that came up or went down between the ask and the answer does not refuse the strand the ask went to.
  Any other caller is refused, naming the way forward: the run's parent orch answers.
- The verb answers an ask in state `pending-post` or `open`, since the mail may have gone out before a crash left the state unset.
  It refuses when `--ask` names no such ask: the run's latest ask has another number, or the run has none, or the ask is already `done`, `timeout`, `no-parent` or `not-sent`.
  An ask still `pending-post` or `open` at or past its deadline is set `timeout` and the answer refused.
- The refusal of an `--ask` that is not the latest, while the latest ask is `pending-post` or `open`,
  names the latest ask's number and says the record may have changed since the named ask;
  the way forward is to review the record again under the latest ask's own request, then answer with `--ask <latest>`.
- Every other refusal of the ask names the way forward:
  a call the parent still wants made goes in with `lym loom decision add <slug> --by parent`.
  The refusal names who runs it as that verb's guard resolves it: the run's parent orch,
  or the operator's own unnamed shell when no parent orch resolves or the resolved one has no name.
- When the round's review is accepted, one step under the per-run lock (D5) copies a `done` report beside the review file,
  as `keepAssistReports` does the assistant's, so it is committed with the round.
  The step runs on a fresh table and on the hand-on path of a restarted row whose review is already accepted
  (`internal/shedadapters/burlerreview.go:257-259`), and copies a report no earlier run of it copied.

Bound: the verb writes only the parent report and the state of the ask `--ask` names; it edits no tracked file and never the decision record.
Its guard checks an unverified strand name and stops only a mistaken call, not a forged one (`internal/loomcli/decision.go:11`).

### D5 — The deadline: `discussion_review_parent_timeout_min`, and a capped wait the chair runs

A second new key, `discussion_review_parent_timeout_min`, default `30`, category `none`, sets the minutes from the ask to its deadline before the cap.
It must be a positive integer; `loomengine`'s config validation refuses `0` or a negative value, naming the key.

The cap follows `CapAssistTimeout` (`internal/loomengine/review.go:64-84`), with room left for the chair to consolidate after the wait:

- The chair's timeout is `review_timeout_min`, or shuttle's `run_timeout_min` when that key is `0`.
- The room is the chair's timeout less shuttle's start allowance, less one chair keep-warm await cap, less a consolidation margin of 5 minutes,
  a named constant in `loomengine`.
- The deadline is the ask time plus the lesser of `discussion_review_parent_timeout_min` and the room.
  A capped deadline logs one Warn naming `discussion_review_parent_timeout_min` and `review_timeout_min`.
- A room of zero or less sends no ask: the round logs one Warn naming both keys and saying to raise `review_timeout_min`,
  and writes the ask record in state `not-sent` with that reason, which the review's line names.
  The next round of the same seed tries again (D2).

The chair waits for the parent with a new verb, `lym loom parent-review await <slug>`, run in the foreground before it consolidates.

- It answers `ready: true` once the ask is `done`, `timeout`, `no-parent` or `not-sent`.
- At or past the deadline with the ask still `pending-post` or `open`, it sets the ask `timeout` and answers `ready: true`.
- Otherwise it waits, waking at most once a second, up to the keep-warm cap `shuttleengines.KeepWarm` gives the chair's strand, then answers `ready: false`, and the chair runs it again.
- Its `ready: true` answer carries the ask's state, the `not-sent` reason when the state is `not-sent`,
  and the parent report's path when it is `done`.
- Bound: any caller may run the await; it writes only `timeout`, and only to an ask at or past its deadline,
  which D4's answer check also does, so it ends no ask earlier than its deadline.

When the chair reaches done with its review written, the review gate's parent coverage entry (D7)
first sets an ask of the round still `pending-post` or `open` to `timeout`, under the lock.
So an ask the chair left without running the await is closed before its review can be accepted, and a late answer to it is refused.
A table that fails before its chair reaches done leaves the ask as it is, for the row's restart to post or await up to the first deadline;
an ask whose row never runs again is closed by D4's deadline check on the first late answer.

The ask's state moves only forward, from `pending-post` to `open` and from either to an end state, under one per-run lock every writer takes,
so an answer and a timeout never both land: the second is refused (answer) or reads the first (await).

On `timeout`, `no-parent` or `not-sent` the chair consolidates without the parent and writes one line in the review saying the parent review did not arrive and why;
that line and the ask record are the record that the round went on without it.

Bound: the capped deadline is the longest the parent can hold a round.

## Constraints

- `PATTERN-config-layers`: both new keys go through `configengine`, with defaults in `internal/loomengine/template.yaml`; no meaning of an existing key changes, so no `hubreconcile` reinterpretation ships.
- `PATTERN-agent-addressing`: the ask goes through creel, via `orchcli.NotifyParent`'s routing; the parent is resolved by the existing parent routing.
- `PATTERN-refusal-way-forward`: D4's refusals name their way forward, and a test reaches each refusal.
- `PATTERN-daemon-wakeups`: D5's wait wakes at most once a second and takes its cap from the keep-warm resolver.
- `PATTERN-lym-files`: the ask record and the parent report live under `.lym`, named through `lymdirs`, and D5's lock path comes from `lock`; the copy beside the review is tracked under `_lym` like the assistant's copy.
- `PATTERN-tolerant-records`: the ask record is Go-written JSON written through `internal/state` and refuses no unknown key.
- `PATTERN-cli-cobra`: `lym loom parent-review` is a cobra subtree of the `loom` module.
- `PATTERN-stencil-ownership`: the parent consolidation part is a stencil read at call time through `stencilstore`.
- `PATTERN-run-records`: the parent report is run state, not a friction note, usage record or project record; the decision record is still written only by its writers.

## Auto-mode assumptions

- The defaults and picks of D2, D4, D5 and D7 are auto-picks; the support log's ledger gives each one's rationale.

## Open risks

- None.

## Acceptance criteria

- D4: `--satisfied` sets the ask `done` with a report that holds no findings; `--findings-file` sets it `done` with the file's findings, each keeping its given severity and class, with ids `P1`, `P2` and so on in the file's order.
- D4: a findings file with no item, an item with no `change`, or a severity or class outside the review format's words is refused, naming the file's shape.
- D4: the answer verb is refused, each refusal naming `lym loom decision add` and who runs it, for a slug with no ask, for an `--ask` that is not the latest ask's number while the latest is in an end state, and for an ask already `done`, `timeout`, `no-parent` or `not-sent`;
  the refusal names the run's resolved parent orch, or the operator's unnamed shell when no parent orch resolves.
- D4: an `--ask` that is not the latest ask's number, while the latest is `pending-post` or `open`, is refused naming the latest number and saying to review the record again before answering it.
- D4: the answer verb is refused for a named caller that is neither the run's parent orch nor the strand the ask went to, and admitted for that strand after the parent orch resolves to another strand, the ask's state `pending-post` included.
- D4: an answer to an ask still `open` at or past its deadline sets the ask `timeout` and is refused, naming `lym loom decision add` and the run's resolved parent orch.
- D5: the await verb answers ready once the ask is answered or already `timeout`; at the deadline with no answer it sets the ask `timeout` and answers ready; before both it answers not ready once its cap runs out.
- D5: the await's ready answer names the ask's state, the `not-sent` reason for a `not-sent` ask, and the report's path for a `done` one.
- D5: with `review_timeout_min` shorter than `discussion_review_parent_timeout_min`, the deadline falls before the chair's timeout by at least the consolidation margin.
- D5: an answer racing the deadline leaves exactly one of `done` or `timeout`, and the losing answer is refused.
- D5: `discussion_review_parent_timeout_min: 0` is refused by config validation naming the key.
- Review: the `internal/loomengine/template.yaml` comments for both new keys say they are settable per run through `run_settings`.
