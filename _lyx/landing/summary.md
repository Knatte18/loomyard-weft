# Discussion-Write: add a seats producer, a chair with advisors

Discussion-Write can now run a second producer: a chair that interviews and writes the decision record and the support log, plus advisors that study the task, the board, the code and PATTERN in parallel and answer the chair's design questions live over the session message channel.
Agents designing together beat one agent asking and answering itself, and an advisor's dialogue with the chair stays out of the parent's context.
The existing single-agent producer is unchanged and stays the default, so a hub that sets nothing behaves as today.

## What changes

- `loom.yaml` gains `discussion_producer`, `single` (the default) or `seats`, which selects the Discussion-Write row's producer.
  Any other value is a config refusal with its way forward, listed in the refusal spec.
- `loom.yaml` gains `discussion_advisors`, a model-spec list with one advisor per entry; the template default is `["sonnet[high]"]`.
  Only the `seats` producer reads it.
  An empty list runs the chair alone.
  The key joins `loomengine.ConfigOpenMaps`, so `lyx config reconcile` and `--set` keep the operator's scalar, null or list spelling whole.
  Every entry is parsed and resolved at load, whatever `discussion_producer` holds; an empty entry beside another, an unparseable entry or an unresolved one is refused naming the key, the entry index and the way forward.
- `internal/loomengine` builds the discussion seat table in Go (`DiscussionTable`): a chair seat (the `discussion` model, outputs the two discussion files, inputs every advisor's notes file) and one advisor seat per entry (output `_lyx/discussion/advisor-<n>.md`).
  A new registry entry in `internal/shedrecipe` (`DiscussionSeats`) runs the table on `MultiLLMProducer` behind the commit decorator, with the row's gates on the chair.
  `internal/loomrecipe` builds the Discussion-Write row with the engine `loom.yaml` selects; the row keeps its name, gates and routing under either producer.
- Two new stencils: `loom-template-discussion-chair` (includes the discussion stencil whole and adds an advisors section) and `loom-template-discussion-advisor`.
  The chair sends every question batch to every advisor and settles after weighing all answers.
  A relied-on advisor answer is recorded under its turn in `## Interview`, and in autonomous mode a pick resting on one is marked advisor-informed in the question ledger, both naming the advisor's strand.
  An advisor writes its notes file as soon as its study is done, answers the chair with evidence and a recommendation, messages only the chair, and mutates no git state.
- The autonomous `mode_rules` text gains one sentence, only when the table has advisors, sending design questions to the advisors.
- `MultiLLMProducer` gains a second constructor taking a per-call table source and a fresh-spawn preparation returning an error; the existing constructor keeps its signature.
  On the fresh path the order is probe, prepare, archive, run; the preparation never runs on resume, and its error archives nothing and starts no seat.
  The entry wires the parent-review store's `PrepareRound` as that preparation when the gate list holds an enabled `parent-review` entry.
- `seatengine.Table` gains a field naming the table values that may be empty; `pattern_directive` and `friction_directive` are listed.
  A listed name that is also a reserved marker is a validation error, and every unlisted value keeps the blank-value refusal.
- `roleOpeningStencils` keys the discussion seats as `discussion-chair` and `discussion-advisor`.
- Docs: `docs/overview.md`, the `shedadapters`, `shedrecipe`, `loomrecipe` and `seatengine` package docs, the shed-recipe spec and the refusal spec.

## Design decisions

- The advisor notes file is committed with the discussion by the existing whole-directory commit and archived on a fresh respawn like the two discussion files.
  No Go code parses it, Plan-Write never reads it, and the discussion gate, `lyx loom decision add`, the Discussion-Review targets and the parent-review brief still read the two discussion files only.
- Every message to the producer goes to the chair, and only the chair is interactive under `discussion_interactive`; the parent's verdict verbs are unchanged.
  A parent-review reject re-prompts the live chair, so the advisors stay up for the rework.
- Failure handling is the seat engine's: a chair death or timeout halts the run, an advisor that fails or dies is reported to the chair once, and every advisor is stopped when the chair ends.
- An earlier design retired the single-agent producer; it was kept instead so the new producer can be tried beside it and switched back by config.
- The template default is one advisor, but the default producer is `single`, so no hub gets advisors until it sets `discussion_producer: seats`.
- In a present `loom.yaml`, an absent `discussion_advisors` takes the template's one advisor because reconcile fills missing keys from the template; null, `[]`, `""` and `[""]` load as zero advisors.

## To check by hand

- An in-flight run parked on Discussion-Write under the old binary is restarted, not migrated: the seat probe looks for a `discussion-chair` strand, so under `seats` it would start a fresh table beside a live old writer.
  Stop such a writer before the new binary drives the run.
- Advisor edits are bounded by stencil text and the agents' settings denies only.
  An advisor can edit the two discussion files or add files under `_lyx/discussion/`, and an edit made while the chair's gate waits reaches the commit with only the Discussion-Review perch ahead of it.
- An advisor can message the run's parent, the orch, and only the orch's judgment keeps it from acting on the message; the advisor strand name `discussion-advisor-<n>` identifies the sender.
- `discussion_advisors` must keep its scalar or null value through `lyx config reconcile`; the open-map entry is covered by tests, but run a reconcile on a real hub once.
- The `claims-rows-gofmt-indent` finding on the chair-stencil claims rows in `contracts/stencils/claims_test.go` was fixed after its review; confirm `gofmt` is clean there.
- The seat channel itself is not exercised by any gate: a live `discussion_producer: seats` run (chair plus one advisor, autonomous and interactive) was not run.
- The integration suite was running when the last run record was written; run `lyx gate test --tags integration ./...` before relying on the change.
