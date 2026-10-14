# Support log — overview-seats

## Interview

Autonomous run: no operator answered, so every turn below is the writer's exploration and its own pick.

### Turn 1 — board and context

Read the board entry and the entries that touch it: `discussion-multi-llm-bench` (bench design with a single-vs-N=0 control arm), `arbiter-when` (arbiter records and stop keys), `report-timing` (reads the same history and pause gaps), `twill-fork-timing` (lens forks spawn after writing, landed in `f0e753ba2`), `model-downgrade` (names `burler-*-chair` roles in its reports), `hibernate-trigger` (hibernates only after a pause), `darn-run-budget` and `orch-split` (no overlap).
`burler-split-budget`, named in the entry as in flight, has landed: Burler runs as `BurlerReview` and `BurlerFix` rows, each a one-seat table.

### Turn 2 — the overview window

The window is `lyx loom checklist` (`internal/loomcli/checklist.go`), drawn for loom and darn runs.
It renders detail blocks per row, then the row list: the duplicate the operator saw.
Time per visit runs from the previous history entry's end, so pauses and sleeps count.
`pauserecord` already holds pause intervals for seven days with `Excluded` to subtract them; the batten overview reads the same record.
Picks: one combined frame (D1); segments by round in run order (D2); paused time subtracted from the pause record (D5).

### Turn 3 — `stuck` for a bounce

`shedengine.Stuck` is the outcome both of a bounce along `OnStuck` and of a writer's gate-failed halt; Burler rows never return Done, so every Burler visit reads `stuck`.
`composeActivity` already writes "bounced to" from the routing call site and notes the routing cannot be inferred from the persisted fields.
Pick: Shed records the target on the Stuck entry's `routed_to`, and the checklist labels it (D4); the persisted outcome and `on_stuck` stay.

### Turn 4 — arbiter and escalation lines

Awaiting verdicts are history entries; arbiter attempts live under `shedrun.ArbiterDir` with a stop key carrying the history length.
Pick: place each attempt after the entry its stop key names, and label awaiting entries `escalated` (D3).

### Turn 5 — seat names

`chair` and `advisor-<n>` are load-bearing in `seatengine` (validation, prompt values, gate and spec choice), in `loomengine.DiscussionWriterRole`, in `burlerengine`'s table builders, in `roleOpeningStencils` and in the `burlercli` help text.
Picks: lead by place, carrying the role prefix alone; later seats named by recipe or config (D6); Discussion advisors named in `loom.yaml` (D7).
The lead keeping the bare prefix leaves every single-writer row's strand name as it is today.

### Turn 6 — retiring `SingleLLMProducer`

Users: Discussion-Write, Plan-Write, Darn-Write, Describe, PR-Rework, and the generic `SingleLLM` row, which no shipped recipe uses.
The board's open question, what single has that seats lacks: the gate (seats has `Table.Gate`), interactivity (`Table.Interactive`), spec sourcing per call (`TableSource`), the fork flag (`Seat.ForkSubagents`), and probe-before-archive (`Engine.Probe`) all exist; the prompt amendment from `prepareFreshSpawn` is the one gap, used by Plan-Write.
Overhead with zero advisors: the hold-deferral predicate counts advisors only, and the notice sender has none to report, so a one-seat table defers and types nothing.
The hub config holds `discussion_producer: twill` with `discussion_advisors: ["sonnet[high]"]`, so moving to the two-axis config without a migration would start a dormant advisor.
Picks: D8, D9, D10, D11.

### Turn 7 — bench

The entry asks for an N=0 run against solo before parking the single producer.
Pick: pin equivalence mechanically (Acceptance criteria: each converted row's lead spec equals the old spec field by field), since this task removes the single producer and an LLM bench is an outcome comparison of the same prompt on the same model.

### Turn 8 — lens review

Every fork returned a report.

- premises, BLOCKING: `PR-Review` is a segment with no Bouncer, so D2 dropped its visits — fixed: D2 defines a review segment as one holding a Bouncer; others render as D1 rows.
- premises / scenarios, MEDIUM: an advisor named `decision-record` takes a lead output path — fixed: notes at `advisor-<name>.md` (D7).
- premises / pattern-fit, LOW: `roleOpeningStencils` needs the `discussion` key remapped and a rule for open-ended advisor roles — fixed in D6.
- premises, LOW: tackler and alignment attempts not covered — fixed: listed under Scope Out.
- premises, LOW: interactivity source differs between spec and table — fixed in the parity criterion.
- goal-scope / scenarios, MEDIUM: round numbering left to Plan, visits before the first Bouncer and gotos unplaced — fixed: rounds count review-row visits as `ResolveRound` does (D2), with a criterion.
- goal-scope, MEDIUM: the `MultiLLM` row's first seat `name` key not covered — fixed in Scope.
- goal-scope, MEDIUM: the bench the entry asks for is dropped unlicensed — fixed: stated as a departure the operator may overrule in Auto-mode assumptions.
- goal-scope, LOW: D6 departs from "operator names each seat" — fixed: stated in D6.
- goal-scope, LOW: parity criterion vs. the Discussion prompt change — fixed in the criterion.
- goal-scope / leanness / scenarios: stale-strand risk unbounded and, per scenarios, false (the attach probe matches output files) — fixed: risk dropped, attach-after-deploy criterion added.
- goal-scope, LOW: `escalated` label unlicensed — rejected: an `awaiting` line inside a past round reads as a wait in progress, and the entry asks for the escalation to show.
- board-fit, MEDIUM: `claimed-scope-edit` may add a single-producer row — fixed: Scope covers every user at plan time.
- board-fit, MEDIUM: the bench note's whole method changes, not only its control arm — fixed in Open risks.
- board-fit, LOW: lens and perch fan wording collide — fixed: D8 doc-comment rule.
- board-fit, LOW: D5 leaned on unbuilt `hibernate-trigger` — fixed: the claim is cut.
- board-fit, LOW: `report-timing` overlap — fixed: note for the plan writer.
- board-fit, LOW: `model-downgrade` names old roles — rejected: already covered by the tokencount risk.
- pattern-fit, MEDIUM: `PATTERN-agent-name` departure unnamed — fixed: Constraints names the entry change and its bound.
- pattern-fit, MEDIUM: `PATTERN-refusal-way-forward` spec rows missing — fixed in Scope and Constraints.
- pattern-fit, LOW: `discussion_advisors` category none now names strands — fixed: one clause in D7.
- pattern-fit, LOW: `routed_to` doc comment — fixed in Constraints.
- pattern-fit, LOW: test-budget file — rejected: the CLAUDE.md rule binds the plan's cards directly.
- scenarios, MEDIUM: a lens key naming a missing fan — fixed: refused at table build (D8, D9), criterion added.
- scenarios, LOW: arbiter stop key beyond history, no arbiter directory, unreadable recipe — fixed in D1, D3 and criteria.
- scenarios, LOW: pause grace counted — fixed: stated in D5.
- scenarios, LOW: a renamed advisor leaves its old notes file — rejected: the lead reads only its current inputs, and a stale notes file is inert.
- leanness, LOW: rejected-alternative arguments in D4 and D6 rationales, the hibernate narrative in D5, the long D10 rationale — fixed: cut.
- leanness, MEDIUM: Auto-mode assumptions restate decisions — fixed: bullets are pointers.
- leanness, LOW: overlapping notes for the plan writer — fixed: trimmed.

## Rejected alternatives

- Two checklist parts with clearer headings, or dropping finished rows from the checklist: the combined view shows what happened and what is left in one place, which neither does.
- Grouping a segment's log by row: loses the order the run took.
- Renaming `shedengine.Stuck` or `on_stuck` to `bounced`: a Stuck verdict is also a writer's gate-failed halt, the value is persisted in every status file, and the recipe key is read by stencils and specs; the label belongs in the display.
- Inferring a bounce from the next history entry: fails for a self-bounce (`Run-Shed` in batten) and contradicts `activity.go`'s own reasoning; recording `routed_to` is one field at the call site that already knows.
- Counting machine sleep through `pauserecord.SleepWatch`: it is live-only, and the pane process may restart and lose it; the hibernate path goes through a pause.
- A `lead` flag on `Seat`: a second rule to validate where table order already marks the lead.
- Naming the lead seat too (for instance `writer`) in a multi-seat table: it changes the lead's strand name with the advisor count and gives the parent-review gate two roles to find.
- A round number in strand names: the overview now shows rounds, and reed already numbers names a live strand holds.
- A parallel `discussion_advisor_names` list beside the model list: two lists that must stay aligned.
- One `discussion_lenses` value under the existing `discussion_fan` key: `discussion_fan` already turns on the Discussion-Review perch's fan, a different thing.
- Keeping `discussion_advisors` category none: with `discussion_producer` gone, a local override or a board task's `run_settings` could start any number of advisor seats, a reach the retired guard key denied and nothing in the task asks for; a guard key keeps today's reach, and the bench's pair configs are worktree files, which may set it.
- Keeping `SingleLLMProducer` for the one-seat rows: two producers is what the task removes.
- An LLM bench of N=0 against solo in this task: see Turn 7.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Which window is "the overview window"? — `lyx loom checklist`, the run's loom-status pane (Turn 2).
- Q2 (auto-pick): Combined view or clearer two parts? — Combined, by round within a segment (D1, D2).
- Q3 (auto-pick): Rename the stored outcome or only its display? — Display, from a recorded `routed_to` (D4).
- Q4 (auto-pick): Show the arbiter how? — One line after the entry its stop key names; awaiting entries read `escalated` (D3).
- Q5 (auto-pick): Paused time? — Subtract the pause record's coverage of the slug; unpaused sleep counts (D5).
- Q6 (auto-pick): Lead marked by place or flag? — Place (D6).
- Q7 (auto-pick): Lead's name with advisors? — The role prefix alone (D6).
- Q8 (auto-pick): Where advisors are named? — `discussion_advisors` entries `{name, model}` (D7).
- Q9 (auto-pick): Lens axis keys? — `discussion_lenses`, `plan_lenses`, guard keys naming a `burler.yaml` fan (D8, D9).
- Q10 (auto-pick): Old configs? — `MigrateConfig` rewrites; dormant advisors dropped unless the producer was seats (D11).
- Q11 (auto-pick): Bench before removing single? — No; field-by-field spec parity tests (Turn 7).
- Q12 (auto-pick): Round in role names? — No (Rejected alternatives).
- Q13 (auto-pick, after lens review): How are rounds numbered? — By review-row visits, as `ResolveRound` numbers the round reports (D2).
- Q14 (auto-pick): Where do advisor notes go? — `_lyx/discussion/advisor-<name>.md` (D7).
