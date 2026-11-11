## Goal

The Discussion-Review round asks the parent orch, and its chair keeps every parent finding

Takes: parent-review-ask-verbs' ask record API, deadline cap, config keys and verbs; NotifyParent's routing; the assistant seat as the model for the consolidation part and the copy beside the review
Delivers: the asking rule and creel ask in Discussion-Burler-Review, signed as <shortname>:LOOM; the parent consolidation stencil and marker; Finding's parent_ids and the review format stencil's key; the parent coverage gate entry; the copy of a done report beside the review; the orch-role section for [discussion-review request]
Exports: the [discussion-review request] creel line, Finding.ParentIDs, the parent coverage gate entry, an orchcli post to a given strand signed as a program

## Scope

The part changes these packages:

- internal/burlerengine (about 500 changed lines)
- internal/shedadapters (about 300 changed lines)
- contracts/stencils (about 90 changed lines)
- internal/orchcli (about 50 changed lines)
- internal/loomcli (about 40 changed lines)
- contracts/orchrole (about 40 changed lines)

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
Each Discussion-Burler-Review round reads the key, which decides only whether the round sends a new ask.
An ask already written in the round, `pending-post` included, is posted, awaited and consolidated whatever the key now reads,
so a key turned off before a restarted review row still ends the round's ask.
With the key on, an asking round asks the parent orch to review the decision record at the same time as the chair's lens forks and any assistant review it.
The parent's findings join that round's review, Discussion-Burler-Fix fixes them in the same round, and the Discussion-Bouncer judges the round as it does today.
No row, pause, extra round or parent decision is added.

Facts the asking rule rests on:

- A new generation starts only when a settled perch's subject changed (`startNextGeneration` in `internal/shedadapters/amendment.go`).
  It archives the run directory and seeds round 1 again, so round numbers restart at 1.
- The Bouncer's clear (`entryClear`, `internal/shedadapters/bouncer.go:458-470`) also archives the run directory and seeds round 1.
- A goto and a resume restart nothing (`internal/shedadapters/bouncer.go:238`, `:367-369`).
  After a goto back to Discussion-Write over an undecided escalated round, the perch halts Awaiting on that round again,
  and a `continue` sends the rewritten record to a fresh round.
- The run status's `history` records producer calls by producer name (`HistoryEntry` in `internal/shedengine/status.go`).
  A run of identical calls folds into one entry whose `LastAt` holds the latest time (`internal/shedengine/run.go`),
  and a goto appends an entry naming its target (`internal/shedengine/goto.go`).

A round's seed is the round-1 focus file the Bouncer's seed wrote (`seedCall`, `internal/shedadapters/bouncer.go:1621-1651`), told apart by its write time.
The ask record stores that file's modification time at the file system's full precision, RFC 3339 with nanoseconds,
and compares it exactly with the file's current modification time.
A fresh `lym loom start`, a new generation and a clear each write a new one;
a goto, a resume and a restarted row keep it.

Which rounds ask, read against the run's latest ask (D3):

- A round asks when the run has no ask yet, or when its seed differs from the latest ask's seed,
  so a fresh start, an amendment and a clear each send their record to the parent.
- A round asks when the run status's `history` holds a `Discussion-Write` entry whose `At` or `LastAt` is after the latest ask,
  so a record rewritten after a goto reaches the parent although no generation starts.
- A round asks when the latest ask has the same seed, was sent in the round before, and ended `done` with findings,
  so the parent can confirm their fixes.
- A round asks when the latest ask has the same seed, was made in an earlier round, and is `not-sent` (D5),
  so a raised chair timeout takes effect at the next round.
- Otherwise the round asks nothing.

A record changed only by `lym loom decision add` or `lym loom decision replace` sends no new ask: those are the parent's or the operator's own calls.

Bound: the parent's answer never gates the round by itself; approval stays the Bouncer's.
This departs from issue #87's "Plan starts only on approval": when the Bouncer approves after Burler-Fix fixed the parent's findings, no later round runs and the parent is not asked to confirm; the fixed record goes to Plan.
A `discussion_max_bounces` of `0` runs no round, so the parent is never asked;
`discussion_auto_accept`, a hard cap or `lym loom circling accept` can end the perch with parent findings open, as with any other finding.
Rationale: a seat in the round costs no extra round, reuses Burler-Fix and the Bouncer, and keeps the parent's load to one read per written record unless it found something.

### D3 — Go asks the parent once per round, through creel

When a round asks (D2), the Discussion-Burler-Review row sends the parent orch one creel line before the review table starts.
It goes by `orchcli.NotifyParent`'s routing, the path PR-Review's parent ask uses.
The line opens `[discussion-review request]` and names the slug, the round, the ask number, the decision record's path in the task worktree,
the deadline as a wall-clock time, and the answer verb (D4) with its `--ask` flag filled in.
The ask lives in a never-tracked ask record under the run's `.lym` directory, outside the run directory, so neither an archive nor a clear removes it.
It is separate from the Bouncer's per-round parent notice.
The ask record holds the run's latest ask: its number, counting up from 1 within the run, its seed and round (D2),
the time it was made, its deadline, its state, and the orch strand the post went to.
A review row finds its round's ask by seed and round, never by the review attempt token.

- The row resolves the destination strand once, by `NotifyParent`'s routing,
  writes the ask record in state `pending-post` with its deadline and that strand, posts to that strand, then sets the state `open`.
  A restarted or resumed row of the same round that finds `pending-post` posts again to the stored strand, accepting a possible second mail;
  one that finds any later state posts nothing and keeps the first deadline.
- The routing falls back to the prime's orch when no orch above the pair is live (`internal/orchcli/notify.go:41-49`).
  A destination with no orch strand recorded, which `NotifyParent` answers with `posted` false and a nil error (`:193-195`),
  sets the state `no-parent` at once with nothing posted, and so does a post error; each is logged, and the round goes on without the parent.
- A dead orch the post still reaches, the prime's included, leaves the ask `open` until the deadline ends it in `timeout` (D5).
- `NotifyParent` reports no receiver (`internal/orchcli/notify.go:47-50`) and signs as the caller's strand when the process carries a non-orch strand name (`:188-218`).
  So the row posts through a variant that takes the resolved strand and signs as the program `LOOM` alone, ignoring any strand claim,
  and the request always comes from `<shortname>:LOOM`.
- An unreadable `discussion_review_ask_parent` is logged and treated as `false` for that round.

Bound: the ask carries only the slug, round, ask number, path, deadline and verb; the parent reads the record itself.

### D6 — The orch's instructions: a fork may do the whole review and answer it

`contracts/orchrole/orch-role.md` gets a section for mail from the hub's LOOM program, `<shortname>:LOOM`, opening `[discussion-review request]`;
D3 signs every such request as that program, whatever strand the review row's process carries.
It tells the orch to hand the review to a one-shot fork: the fork keeps the orch's knowledge of the run, reads the decision record and the run's board entry and settled calls, and answers itself, so the review stays out of the orch's main thread.
Such a fork may run two writing verbs, both for the slug the request names: `lym loom decision add <slug> --by parent` for a call it wants made rather than a fix, before its answer,
and `lym loom parent-review answer` with the request's `--ask`.
The section gives the findings file's shape (D4).
The section states this as an exception to the forks-only-read rule.
The section names D1's assistant keys as the way to add an agent review, and both D2 and D5 keys with their `run_settings` override.

Fact the fork rests on: `lym loom` verbs carry no fork check, and the caller guard reads the caller's strand name, which an in-session fork shares with the orch, so the fork passes D4's guard.
Bound: the fork's reach equals the orch's.
A fork of an orch that D4 admits only as the strand the ask went to cannot run `decision add`,
whose guard admits only the parent resolved at call time.
The guards check an unverified strand name and stop only a mistaken call (`internal/loomcli/decision.go:11`),
so the fork can run every orch-guarded verb on every run whose parent resolves to the orch,
`lym loom decision replace` and `lym loom decision store` included.
Its limit to the two verbs and the slug is instruction text alone.
Rationale: the operator asked that a busy orch can keep the review off its main thread.

### D7 — The chair keeps every parent finding

The review step gets a parent consolidation part beside the assistant's part.
It is filled whenever the ask record holds an ask for the round's seed and round, `not-sent` included,
whatever `discussion_review_ask_parent` now reads.
It tells the chair to wait with D5's verb.
On `done`, the chair reads the parent report at the path the await names and keeps every item in it as a finding tagged `origin: parent`,
with the item's `severity`, `BLOCKING` when the item gives none;
on any other state it writes D5's line.
A parent finding keeps the item's `class` when it is `design`, `scope` or `consistency`; a `decision` class or none becomes `design`, so no parent finding escalates back to the parent.
The chair may merge a parent finding with a duplicate of its own, and the merged finding keeps `origin: parent`;
it never moves one to `## Rejected`.
Each finding tagged `origin: parent` names the report ids it covers in a `parent_ids` list, a finding key the review format gains.
The chair's verdict reflects open parent findings like any other kept finding.

The review's gate gains a parent coverage entry, listed after the review-parse entry (`ReviewGateEntry`),
so it runs only at an arrival whose review parses.
It is in the gate of a round whose ask record holds an ask for its seed and round.
Under the per-run lock it closes a still-open ask (D5);
then, when the ask is `done` with findings, it fails naming every report id that no `origin: parent` finding's `parent_ids` lists.
A failure re-prompts the chair.
The entry never passes on its cap, so a review that still leaves an id uncovered fails the gate.
The review row then fails with the gate's reason, naming the missing ids, as on any failed review gate,
and the row's restart runs a fresh table whose chair finds the ask closed and the report `done`.

Rationale: the parent is the operator's delegate, so a reviewer agent does not overrule it, and the coverage entry holds the chair to that in Go.

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

## Acceptance criteria

- D1: a task whose `run_settings` sets `discussion_assist_reviewer_model: fable[high]` over a hub that sets none runs its Discussion-Review round with an assistant seat on that model; existing coverage may meet it.
- D2: with `discussion_review_ask_parent` off, a Discussion-Burler-Review round sends no `[discussion-review request]`; with it on in the hub, or on in a task's `run_settings` over a hub that leaves it off, the run's first round sends one.
- D2: a task's `run_settings` setting `discussion_review_ask_parent: false` over a hub that sets it `true` sends no ask.
- D2: a later round of the same seed asks again when the round before's parent answer held findings or the ask was `not-sent`, and asks nothing when it was satisfied, `timeout` or `no-parent`.
- D2: the first round of a new generation, whose round number is 1 again, asks whatever the previous generation's last parent answer was.
- D2: after a satisfied parent answer, a goto back to Discussion-Write over an undecided escalated round, followed by a `continue`, makes the next round ask about the rewritten record.
- D2: with the key turned off after a round's ask, a restarted review row of that round sends no new ask, posts an ask it finds `pending-post`, and its chair awaits and consolidates the round's ask.
- D3: the ask line names the slug, the round, the ask number, the decision record's path, the deadline and the answer verb with its `--ask` flag.
- D3: a review row restarted in the same round, with a new attempt token, sends no second ask and keeps the first deadline once the ask is `open`, and posts again when it finds the ask `pending-post`.
- D3: a Bouncer clear after a `done` ask seeds round 1, which sends a new ask and keeps none of the old report's findings.
- D3: with no orch strand recorded at the destination, nothing is posted, the ask is set `no-parent`, the round goes on, and the review records that the parent review did not arrive.
- D3: a review row whose process carries a non-orch strand name posts the request signed as `<shortname>:LOOM`.
- D3: an ask posted to an orch that never answers is set `timeout` at the deadline.
- D4: a `done` parent report is copied beside the review file once the round's review is accepted.
- D4: a row restarted after its review was accepted, with a `done` report not yet copied, copies it, and a later answer to the round's ask is refused.
- D5: with a chair timeout that leaves no room, the round sends no ask, sets the ask `not-sent`, logs a Warn saying to raise `review_timeout_min`, and the review's line names the reason.
- D5: a round whose ask ends in `timeout` writes the review line saying the parent review did not arrive, naming `timeout` as the reason.
- D5: an ask still `open` when the chair reaches done with the review written is set `timeout` before the review is accepted, and a later answer to it is refused;
  after a table that failed before its chair reached done, the restarted row's chair still awaits it up to the first deadline.
- D7: in a round whose parent report holds findings, a review that leaves a report id in no `origin: parent` finding's `parent_ids` fails the parent coverage entry, naming the missing ids, and is not accepted; one that covers every id passes it.
- D7: a round whose ask is `not-sent`, with `discussion_review_ask_parent` off, still fills the parent consolidation part.
- Review: D6 — `contracts/orchrole/orch-role.md` has a section for a `[discussion-review request]` line from the hub's LOOM program, that hands the review to a one-shot fork, lets that fork run `lym loom decision add --by parent` and `lym loom parent-review answer` for the named slug only, gives the findings file's shape, and names the D1 assistant keys and the D2 and D5 keys.
- Review: D7 — the parent consolidation stencil tells the chair to wait with the await verb, keep every parent finding with `origin: parent` and the `parent_ids` it covers, keep a merged finding's `origin: parent`, give a finding with no severity `BLOCKING`, map `decision` to `design`, and never reject one.
- Review: D7 — the review format stencil names the optional `parent_ids` finding key, and `burlerengine`'s `Finding` reads it.
