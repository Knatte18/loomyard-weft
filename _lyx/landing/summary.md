# Discussion-Write: add a seats producer, a chair plus advisors

Adds a second producer for the Discussion-Write row: a chair writes the decision record and support log, and advisors study the task, the board, the code and PATTERN in parallel and answer the chair's design questions live over the session message channel.
The existing single-agent producer is unchanged and stays the default, so a hub that sets nothing behaves as before.

## What changes

- `loom.yaml` gains `discussion_producer` (`single` or `seats`, default `single`), which selects the producer the Discussion-Write row runs.
  An unknown value is a config refusal.
- `loom.yaml` gains `discussion_advisors`, a list of model-specs with one advisor per entry, read only by the seats producer.
  The chair's model stays the `discussion` key.
  An empty list runs the chair alone; an absent key, null, `[]`, `""` and `[""]` all load as zero advisors, and an entry that is empty beside another entry, fails to parse or fails to resolve is refused at load, naming the key, the index and the way forward.
  The template default is `["sonnet[high]"]`, which a present `loom.yaml` picks up when the key is absent.
  The key is an open-map key, so `lyx config reconcile` and `--set` keep the operator's spelling whole.
- `internal/loomengine` builds the seat table in Go (`discussiontable.go`) and `shedrecipe` gains a `DiscussionSeats` registry entry that runs it on `MultiLLMProducer` behind the commit decorator.
  The table is rebuilt on every call, so stencils and directives are read at call time.
  `loomrecipe` builds the row with the engine `discussion_producer` selects; the row name `Discussion-Write` is unchanged.
- `MultiLLMProducer` gains a second constructor taking a per-call table source and a fresh-spawn preparation; the existing constructor keeps its signature.
  The preparation runs after the probe finds no live chair and before the archive, never on the resume path, and an error from it archives nothing and starts no seat.
  The entry uses it to keep the parent-review preparation as the single-agent path has it.
- `seatengine.Table` gains a list of value names that may be empty, for `pattern_directive` and `friction_directive`; only a listed name may be blank, and a listed reserved marker is a validation error.
- Two stencils: `loom-template-discussion-chair` (an include on the existing discussion stencil, which keeps every claim it pins) and `loom-template-discussion-advisor`.
  An advisor writes its notes to `_lyx/discussion/advisor-<n>.md` as soon as its study is done, answers the chair with evidence, messages only the chair and mutates no git state.
  The chair records each advisor answer it relied on under its turn in `## Interview`, and in autonomous mode marks an advisor-informed pick in the question ledger, both naming the advisor's strand.
  The autonomous `mode_rules` text gains one sentence when advisors exist.
- `docs/overview.md`, the `shedadapters`, `shedrecipe`, `loomrecipe` and `seatengine` package docs, the recipe spec and the refusal spec's loom section describe the two producers and the keys.

## Design decisions

- Two producers side by side rather than a replacement, so the seats producer can be tried and switched back by config.
  `discussion_advisors` is read only when `discussion_producer` is `seats`, but its entries are resolved at load whatever that key holds.
- The advisor notes file is committed with the discussion files, read by no parser and never by Plan.
  The discussion gate and the parent-review brief still read only the two discussion files.
- Every message to the producer goes to the chair, and in interactive mode the operator's turns go to the chair alone.
- The role map in `contracts/stencils` keys seats by strand role: `discussion-chair` and `discussion-advisor`.
  The token report needs no change and shows the seats as separate rows.
- Not bounded here: an advisor ignoring its brief can still edit the decision record or add files under `_lyx/discussion/`.
  An edit made while the chair's gate waits skips the self-check and the parent review, leaving the Discussion-Review perch as the one check before Plan.
- An in-flight run parked on Discussion-Write under the old binary is not migrated.
  With `discussion_producer: seats`, stop that run's writer before the new binary drives it.

## To check by hand

- `discussion-seats-row-bypasses-gate-quiescence-guard`: the `DiscussionSeats` row runs on `MultiLLMProducer`, so check that the gate-quiescence guard in `internal/loomrecipe` (`gatequiescence_test.go`) still covers the row's chair gate.
- `card7-call-test-conflicts-with-gate-once-fake`: the `Call` test of the `DiscussionSeats` entry in `internal/shedrecipe/entries_discussionseats_test.go` runs against the fake seat runner in `internal/testkit/shedfake`, which now evaluates a table's gate once on a done chair; check the test and the fake agree.
- `lyx gate test --tags integration` was started for `internal/shedrecipe`, `internal/loomcli`, `internal/testkit/envkit`, `internal/battencli`, `internal/shedbuild`, `internal/loomshed`, `internal/loomrecipe` and `contracts/specs` but its result was not recorded; run it.
- No `llm`-tier test covers the new table; run the seat engine's `llm` smoke by hand to prove the chair-advisor channel, then run a Discussion with `discussion_producer: seats` and one advisor.
- Whether the one-advisor default is worth the tokens: compare the token report's `discussion-advisor` row against earlier Discussion-Review round counts.
- `internal/loomrecipe/seam_enforcement_test.go` allowlist gained `internal/loomshed`, because `loomrecipe` now imports it; confirm the import is wanted.
