# Discussion-Write: add an opt-in chair-and-advisors producer

## What changes

The Discussion-Write row can now run a second producer: a chair that interviews and writes the decision record and support log, while advisors study the task, the board, the code and PATTERN in parallel.
The chair asks its design questions over the session message channel and the advisors answer live.
The chair alone writes the two discussion files, ends the step and takes the gates.
Each advisor writes a notes file, `_lyx/discussion/advisor-<n>.md`, beside the discussion files; no Go code parses it.

The new producer is opt-in.
`loom.yaml` gains two keys:

- `discussion_producer`: `single` (the default) or `seats`.
  The default keeps today's single-agent producer, so a hub that sets nothing behaves as before.
- `discussion_advisors`: a list of model-specs, one advisor per entry, read only under `seats`.
  An empty list runs the chair alone.
  The template default is one `sonnet[high]` advisor.

The row keeps its name, its engine name, its gates and its routing under either producer.
The single-agent path is unchanged and stays as the reference and the fallback.

## Design decisions

- **Seats run on `MultiLLMProducer`.**
  A new `DiscussionSeats` registry entry builds it behind the commit decorator, with the row's gates on the chair.
  `loomrecipe` swaps the row's engine to `DiscussionSeats` when the config selects `seats`, and `RecipeEngines` includes that engine so the cross-consumer coverage guard sees it.
- **The table is built in Go.**
  `internal/loomengine`'s `DiscussionTable` returns the `seatengine.Table` from the told location, config and model registry.
  The entry receives it through a per-call closure, so stencils and directives are read at call time.
- **Two additive seams on `MultiLLMProducer`:** a per-call table source and a fresh-spawn preparation that returns only an error.
  On the fresh path the order is probe, prepare, archive, run; the preparation never runs on resume, and an error from it archives nothing and starts no seat.
  The entry wires the parent-review preparation when the gate list holds an enabled `parent-review` entry.
- **`seatengine.Table` names its optional values.**
  A listed name may be empty and is passed to the stencil fill as an optional marker.
  The discussion table lists `pattern_directive` and `friction_directive`.
  Every unlisted value keeps the blank refusal, and a listed name that is a reserved marker is a validation error.
- **Stencils.**
  `loom-template-discussion-chair.md` includes the existing discussion stencil whole and adds an advisors section.
  `loom-template-discussion-advisor.md` is new; it writes only its notes file, messages only the chair and never mutates git.
  They are registered under the `discussion-chair` and `discussion-advisor` roles.
  The autonomous `mode_rules` text carries an advisor sentence only when the table has advisors.
- **Config load.**
  An absent key, null, `[]`, `""` and `[""]` each load as zero advisors in memory.
  In a present `loom.yaml`, an absent key takes the template's one advisor, because missing keys are filled from the template before the unmarshal.
  An entry that is empty beside another entry, fails to parse or fails to resolve is refused at load, naming the key, the index and the way forward.
  `discussion_advisors` joins `ConfigOpenMaps`, so `lyx config reconcile` and `--set` keep the operator's value whatever its spelling.
- **Parked runs.**
  A run parked on Discussion-Write under the old binary is restarted, not migrated.
  Under `seats`, the probe looks for a `discussion-chair` strand and would start a fresh table beside a live `discussion` writer, so the operator stops such a writer first.
  No code detects this.
- A test-support change: the fake seat runner evaluates a table's gate once on a done chair.
  `internal/loomrecipe/seam_enforcement_test.go` allows the new `internal/loomshed` import.
  The `internal/loomengine` row in `cmd/lyx/testdata/test-budget.yaml` was raised.

Docs updated: `docs/overview.md` (producer paragraph and `loom.yaml` key list), the package docs of `shedadapters`, `shedrecipe`, `seatengine` and `loomrecipe`, and the shed-recipe and refusal specs.

## Check by hand

- The integration tier (`lyx gate test --tags integration ./...`) was not confirmed green: the run record shows an integration run of `shedrecipe`, `loomcli`, `envkit`, `battencli`, `shedbuild`, `loomshed`, `loomrecipe` and `contracts/specs` still in a background shell when the turn ended.
  Run the full tiers before pushing.
- No `llm`-tier test covers the discussion table; the seat engine's existing smoke covers the channel.
  Run a real `discussion_producer: seats` Discussion once to see the chair-advisor dialogue work.
- `card7-call-test-conflicts-with-gate-once-fake`: the `DiscussionSeats` entry's `Call` test in `internal/shedrecipe/entries_discussionseats_test.go` against the fake seat runner that evaluates the gate once; the fix was not re-reviewed.
- `discussion-seats-row-bypasses-gate-quiescence-guard`: the Discussion-Write row on `DiscussionSeats` and `internal/loomrecipe/gatequiescence_test.go`; confirm the guard still covers the writer-engine list for the seats engine, since the fix was not re-reviewed.
- An advisor's study is paid whether or not the chair asks it anything; compare the token report's `discussion-advisor` row against earlier runs' Discussion-Review round counts.
- An advisor edit to the decision record while the chair's gate waits skips the self-check and the parent review; the Discussion-Review perch is the one check left before Plan.
