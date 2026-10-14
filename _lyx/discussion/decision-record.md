# Decision record — overview-seats

## Goal

Three changes to the seat and producer layer and to how a run shows in its overview window:

1. The loom checklist pane (`lyx loom checklist`, loom and darn runs) becomes one combined view: recipe rows in order with their log lines under them, review segments grouped by round in run order, the arbiter shown in the round it decided, a bounce no longer read as `stuck`, and paused time not counted.
2. Seat names: a lone seat carries no seat name, and the seats of a larger table carry the names the recipe or config gives them; `chair` and `advisor-<n>` are never shown, except that a `discussion_advisors` entry migrated by D11 carries `advisor-<n>` until the operator renames it.
3. `SingleLLMProducer` goes: every LLM writer row runs on `MultiLLMProducer` over `internal/seatengine`, with zero advisors unless configured, and Discussion's and Plan's `*_producer` switches become two axes, advisors and lens forks.

## Scope

In:

- `internal/loomcli/checklist.go` rendering, and what it reads: the shed status history, the run's recipe, the run's arbiter attempts and the pause record.
- `internal/shedengine`: the Stuck entry that takes its row's `OnStuck` edge records its target.
- `internal/seatengine` naming and table rules; `internal/shedadapters` `MultiLLMProducer` gaining what `SingleLLMProducer` alone has; `SingleLLMProducer`, `NewSingleLLMProducerGated`, `SpecSource` and the generic `SingleLLM` registry row removed.
- Every row that runs on `SingleLLMProducer` when the plan is written moves to a seat table; today those are Discussion-Write, Plan-Write, Darn-Write, Describe and PR-Rework, and a row `claimed-scope-edit` adds before then joins them.
- The `DiscussionSeats` registry row and `loomrecipe`'s `applyDiscussionProducer` engine swap removed, since Discussion-Write always runs on seats.
- The `MultiLLM` registry row: its first seat drops the `name` key, every later seat keeps it, and `internal/shedrecipe/testdata/multillm-recipe.yaml` follows.
- `loom.yaml` keys: `discussion_producer` and `plan_producer` retired, `discussion_lenses` and `plan_lenses` added, `discussion_advisors` entries named; `loomengine.MigrateConfig` rewrites old files.
- Burler's review and fix halves (`internal/burlerengine`) take the lone-seat role, so their strands read `burler-review` and `burler-fix`.
- Docs in the same commit: the `doc.go` of each touched package, `docs/overview.md` where it names the producers, the chair stencil or `discussion_producer`, `contracts/specs/shed-recipe-spec.md`, a `contracts/specs/refusal-spec.md` row for each new refusal, and the `burlercli` help text that names `burler-review-chair`.

Out:

- The batten runs overview (`lyx batten overview`) and `lyx loom status` output: unchanged.
- The persisted outcome value `stuck`, `shedengine.Stuck` and the recipe key `on_stuck`: unchanged.
- The tackler's and the alignment agent's attempts in the checklist: the entry names the arbiter only.
- The Bouncer row and the `shedadapters.Shuttle` seam: Bouncer keeps running its passes through `Shuttle.Run`, and the Burler rows keep their attach probes.
- A round number in strand or role names.
- An LLM bench of the zero-advisor table against the single writer (see Auto-mode assumptions).
- Measurement reports under `docs/research/` that name `burler-review-chair` or `burler-fix-chair`: they record past runs and stay as written.

## Decisions

### D1 — One combined checklist frame

The frame lists the recipe rows in recipe order, and each visited row's log lines (start, time, outcome, output, gate attempts) stand indented under its own box line.
The separate detail blocks above the row list go, so no finished row appears twice.
A row never reached stays one bare `[ ]` line.
History-only rows (not in the recipe) follow the recipe rows, as today, marked `(not in recipe)`.
With the recipe unreadable, the frame shows `recipe unreadable`, then every visit line in history order, each naming its row.
The state line and the wait line stay last.

Rationale: the operator read the two parts as duplicates; one view in recipe order shows both what happened and what is left.

### D2 — A review segment renders as one block, by round, in run order

A review segment is a recipe `segment` that holds a Bouncer row; a segment without one, such as `PR-Review`, renders its rows as D1 rows.
A review segment's rows render as one block at the place of its first row: a heading line with the segment's name, its box, its time and its round count, then one sub-heading per round, then that round's visit lines in the order they ran, each naming its row.
A review-row entry belongs to the round whose review file its output names, so a hand-on entry (the highest complete round still unjudged, or a restart over an accepted review, both returning a BudgetExempt Stuck without spawning) joins the round its output names, opening it only when no earlier entry has.
Round n holds the review-row entries of round n, the Bouncer entry that bounced into the first of them, and the fix entries after them, which is the order bouncer, review, fix.
A goto or reroute into the review row opens a new round, with no Bouncer line, only when that visit runs a review.
A goto history entry naming a row of the segment carries no output, so it stands as the last line of the round in progress, or before round 1 when none has opened; the review visit after it opens the next round by its output.
The checklist maps an entry's output to its round number through `shedadapters`, the package that owns the round file names, never by a second parse of its own.
A Bouncer entry that routes nowhere (awaiting, or a halt) stands as the last line of the round in progress, followed by any arbiter line D3 places after it; when no round has opened yet, it stands before round 1.
Only a Bouncer Done that ends the segment stands as the block's last line, outside any round.
The segment's box is the box of whichever of its rows is current, else done when any row was visited, else not reached.

Rationale: grouping by row lost the order the run took (bouncer, review, fix, bouncer, ...); grouping by round in run order keeps it.
The segment names come from the run's recipe, which the checklist already parses from the seed's recipe name.

### D3 — Arbiter attempts and escalations show in their round

An `awaiting` history entry shows its outcome as `escalated`.
Each arbiter attempt in the run's arbiter directory (`shedrun.ArbiterDir`) shows as one line placed after the history entry its stop key names (`ArbiterStopKey.HistoryLength`), or at the end when that entry does not exist yet: its spawn time, `arbiter attempt <n>` and its verdict.
The verdict comes from the attempt's envelope (`loomengine.ReadArbiterEnvelope` at `loomengine.ArbiterEnvelopePath`), never from the agent's `verdict.yaml`, so a hand-on the verb concluded itself shows too: the line shows the envelope's verdict word, followed by its cause in parentheses when `Cause` is set.
With no envelope, the line reads `running` while the attempt's `Deadline` lies ahead and `no verdict` after it; an envelope that cannot be read shows `verdict unreadable` and never breaks the frame.
An attempt whose record cannot be read shows as `arbiter attempt unreadable (<directory name>)` at the end of the frame, after the history-only rows and before D1's state and wait lines, since its number and stop key, and so its segment, are unknown; it never breaks the frame; a run with no arbiter directory shows no arbiter line.
The checklist reads each attempt directory on its own (`loomengine.ReadArbiterAttempt` per subdirectory of `shedrun.ArbiterDir`), read-only and without a lock, so one bad attempt leaves the others drawn.

Rationale: round 3's circling escalation and its `resume` decision did not show at all.

### D4 — A bounce reads `bounced`, recorded by Shed

When Shed takes a Stuck verdict's `OnStuck` edge, it writes the target on that Stuck history entry's `routed_to` field, which today only reroute entries carry.
The checklist shows a Stuck entry with `routed_to` set as `bounced`, and a Stuck entry without it (a halt, or an entry written by an older binary) as `stuck`.
The stored outcome stays `stuck`, so every reader of the status file, the bounce-budget count (`episodeStuckCount`) included, sees the same outcome as today; `episodeStuckCount` keeps reading `routed_to` only on reroute entries.
Two folded calls are the same call only when their `routed_to` agree.

Rationale: `composeActivity` already says "bounced to" from the one call site that routes; recording the same fact on the entry lets a reader tell a bounce from a halt without inference, which `activity.go` notes cannot be done from the other persisted fields.

### D5 — Paused time is not a row's time

A visit's time is its wall span less the pause record's coverage of the pair's slug over that span (`pauserecord.ReadCoverage`, `pauserecord.Excluded`), the open visit included.
A live entry counts as paused from its `PausedAt`, its grace period included.
A row's time and a segment's time are the sums of their visits' net times.
A pause record that cannot be read counts nothing as paused and never breaks the frame.
A machine sleep with no pause entry still counts as row time.
A pause resumed more than `pauserecord.Retention` before the frame is drawn has left the record, so its span counts as row time.

Rationale: Plan-Bouncer read 22h18m for a 0-minute row because a pause and the hibernate after it counted.
The pause record already holds every pause and resume of the last seven days (`pauserecord.Retention`).

### D6 — Seat names: the first seat is the lead and carries the role prefix alone

The table's first seat is the lead, which the engine calls the chair internally; no flag marks it, and no seat is named `chair`.
The lead's strand role is the table's role prefix alone, so a lone seat's role reads `burler-review`, `discussion` or `plan`.
Every later seat has a name the table builder takes from the recipe or config, and its strand role is `SeatRole(prefix, name)`.
This departs from the entry's "the operator names each seat": the lead is named by its row's role, the operator names the others.
`seatengine.RoleChair`, `RoleAdvisor` and `AdvisorName` go.
`Table.Validate` requires: at least one seat; the lead carries no name; every later seat a non-empty name, unique in the table, that itself passes the agent-name role grammar (a letter first), as its formed role must, so no formed role reads as reed's numbered form of the lead's role (`agentname.MatchesRole`); and, when the table has more than one seat, a lead stencil that includes `seat-directive-chair`.
The `chair_name`, `seat_name` and `advisor_names` markers carry strand names, as today, so the seat directives need no new marker.
`roleOpeningStencils` in `contracts/stencils/stencils.go` keys the Burler leads as `burler-review` and `burler-fix`, maps `discussion` to `loom-template-discussion-chair`, drops `discussion-chair`, and keeps `discussion-advisor` as the nominal key of the advisor stencil, since advisor roles are open-ended.

Rationale: the operator wants no seat name on a lone seat and operator-chosen names otherwise; naming the lead by the row's role keeps the strand names of today's single-writer rows unchanged and makes the parent-review gate's writer role one value for every advisor count.

### D7 — Discussion advisors are named in `loom.yaml`

`discussion_advisors` becomes a list of maps, each `{name, model}`: `name` is the advisor's seat name and `model` its model-spec.
The template default of `discussion_advisors` becomes the empty list.
The key becomes a guard key, as D8 makes `discussion_lenses`, so it starts advisors only where `discussion_producer` could: the hub-wide file or a worktree's own config, never a local override or a task's `run_settings`.
An entry with a missing, blank or duplicate name, a name that fails D6's seat-name rule, or an unresolvable model-spec, is refused at load, naming the entry and the way forward.
An advisor's notes file is `_lyx/discussion/advisor-<name>.md`, so no name can take a lead output's path, and `DiscussionAdvisorNotes` takes the name.

Rationale: config is where the operator chooses advisors, so it is where they are named.

### D8 — Discussion's two axes replace `discussion_producer`

`discussion_producer` is retired.
`discussion_advisors` (D7) is the first axis: none runs the lead alone, deciding each question itself.
`discussion_lenses` is the second: a fan name from `burler.yaml`, today's `twill-discussion`, whose lenses the lead forks itself for once its record is written and its self-check passes; empty turns the forks off.
Its doc comment says it names the writer's lens fan, not the Discussion-Review perch's `discussion_fan`.
A name with no fan or no lenses in `burler.yaml` (whose fans fall back per name to the embedded template) is refused at table build, naming the key and `burler.yaml`.
Only the lead forks: the table sets `ForkSubagents` on the lead seat alone and renders the twill directive into the lead's prompt alone.
Both axes combine: advisors with lenses on is the bench's round-4 combination.
Its template default is empty, matching today's `single` default.
`discussion_lenses` is a guard key, as `discussion_producer` was and as D7 makes `discussion_advisors`, so the two axes together bound no more than the key they replace: only the hub-wide file or a worktree's own config sets either, so no local override or `run_settings` can start advisors, turn the lens forks off or change their fan.
The Discussion lead always opens with the stencil that wraps the single writer's prompt and the chair's seat directive (`loom-template-discussion-chair`), whose text already covers a lead with no advisors.

Rationale: the operator's two-axis model; zero advisors with the fan on is today's twill, the hub's setting.

### D9 — Plan-Write's lens axis replaces `plan_producer`

`plan_producer` is retired and `plan_lenses` added, a guard key naming a `burler.yaml` fan, today's `twill-plan`, refused as D8's is when the fan is missing; empty turns the forks off, and empty is its template default, matching today's `single` default.
Plan-Write's table has one seat: Plan has no advisor key.

Rationale: the same axis for the other writer that runs twill today; adding advisors to Plan is not asked for.

### D10 — `MultiLLMProducer` takes over every LLM writer row

Each row in Scope builds a seat table from its row's builder in place of a `shuttleengine.Spec`, through `NewMultiLLMProducerSourced`, keeping its gates, commit decorator, model, timeout, interactivity, skills, segment, output files and fork authorization.
Every such row but Discussion-Write builds a one-seat table, and its stencil does not include a seat directive.
`MultiLLMProducer`'s fresh-spawn preparation returns a prompt amendment, as `SingleLLMProducer`'s does, appended to the lead's prompt on the fresh path only, so Plan-Write's rotation still tells the new session where it moved the stale files.
A one-seat table defers no held turn end and types nothing into the lead: its notice sender has no advisor to report on.
`SingleLLMProducer`, `NewSingleLLMProducer`, `NewSingleLLMProducerGated`, `SpecSource`, the generic `SingleLLM` registry row and the `DiscussionSeats` row are removed; the `Shuttle` seam stays for Bouncer and the Burler rows' probes.
`loomengine.DiscussionWriterRole` returns the lead's role, `discussion`, whatever the advisors.

Rationale: one LLM producer instead of two, less code; the prompt amendment is the one thing the seat engine lacks.

### D11 — Old config files migrate on reconcile

`loomengine.MigrateConfig` rewrites a present `loom.yaml`, reporting one line per rewrite:

- `discussion_producer: twill` becomes `discussion_lenses: twill-discussion`; `single` and `seats` become `discussion_lenses: ""`.
- With `discussion_producer: seats`, each string entry n of `discussion_advisors` becomes `{name: advisor-<n>, model: <entry>}`, and the report line asks the operator to rename them.
- With `discussion_producer: single` or `twill`, `discussion_advisors` is set to the empty list, written explicitly even when the document set no list, since those advisors were never read: a lower layer's list then stays off where this document turned it off.
- With `discussion_producer: seats` and no `discussion_advisors` of its own, the document is left as it is, and a report line says its advisors came from a lower layer whose migration may have emptied them and asks the operator to set its named list.
- `plan_producer: twill` becomes `plan_lenses: twill-plan`; `single` or empty becomes `plan_lenses: ""`.

`MigrateConfig` rewrites one document at a time, while whether a list was live depended on the producer the layers resolved to; the two `discussion_producer` rules above keep each document's own choice in force over lower layers.
The `discussion_producer`-keyed rewrites of `discussion_advisors` apply only to a document that sets `discussion_producer` itself.
A sparse worktree file that sets `discussion_advisors` as string entries without `discussion_producer` keeps those entries unmigrated.
The migration runs where every loom config migration runs, under `hubreconcile.Ensure` after a binary change.
A string entry still present after migration in a worktree-layer file is refused at load with the way forward: rewrite the entry as `{name, model}`.
A local layer or a board task's `run_settings` cannot set `discussion_advisors` at all, since `configengine.ValidateLocal` refuses a guard key there (D7).

Rationale: the hub runs `discussion_producer: twill` with `discussion_advisors: ["sonnet[high]"]`; without the migration the dormant advisor would start on the next Discussion.

## Constraints

- `PATTERN-agent-name`: the module that spawns a seat still owns its role prefix as a constant; this task changes the entry so that a later seat's suffix may come from the recipe or config, bounded by the agent-name role grammar and table uniqueness, and the background file says so in the same commit. Roles are formed only through `SeatRole`, and reed forms the strand name once.
- `PATTERN-refusal-way-forward`: every new refusal (D6's table rules reached through the `MultiLLM` row and Discussion-Write, D7's entry checks, D8's and D9's missing fan, D11's string entries) names its way forward, lands its `refusal-spec.md` row and has a test.
- `PATTERN-tolerant-records`: `routed_to` on a Stuck entry is an added meaning of an existing optional field, whose doc comment changes with it; an older binary ignores it, and an entry without it reads as today.
- `PATTERN-told-geometry`: the seat engine and the table builders take every path from their geometry; the checklist, a CLI verb, resolves the pause record and the arbiter directory through `lyxcwd`, `hubgeom` or `standalonegeom` and `shedrun`.
- `PATTERN-shed-recipe-registry`: the `SingleLLM` and `DiscussionSeats` rows leave the one registry map; no `init()` or `Register` appears.
- `PATTERN-config-layers`, `PATTERN-binary-change-reconcile`: the new keys are declared in `loomengine`'s spec and the old ones migrated through the module's `Migrate` hook, reached from `hubreconcile.Ensure`.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`: every converted row's opening stencil keeps both markers, which the seat engine fills; `roleOpeningStencils` lists each lead role.
- `PATTERN-spawn-color`, `PATTERN-role-skills-typed`: segment and skills ride the table and seat, as they rode the spec.
- `PATTERN-completion-signal`: the producer maps the lead's outcome as shuttle reports it, and judges nothing itself.
- `PATTERN-wait-mark-display-only`: the checklist reads no wait mark.
- `PATTERN-documentation-lifecycle` and the repo rule "docs land in the same commit": every touched package's `doc.go` and `docs/overview.md` change with the code.
- `PATTERN-test-economy`, `PATTERN-test-speed`: checklist tests stay pure frame tests over a status value; seat-naming tests extend the existing table tests.
- `PATTERN-comment-line-breaks`, and semantic line breaks in every `.md` file.

## Auto-mode assumptions

- No LLM bench of the zero-advisor table against the single writer runs before the single producer goes, though the entry asks for one; field-by-field spec parity (Acceptance criteria) stands in for it. This is a departure the operator may overrule.
- D6: the lead of a multi-seat table is named by its row's role, not by the operator.
- D3, D4: only the checklist shows `bounced` and `escalated`.
- D5: a machine sleep outside a pause counts as row time, and so does a pause that has passed out of the record's retention.
- D7, D8, D9: `discussion_advisors` and the lens keys are guard keys.

## Open risks

- Removing the single producer and `discussion_producer` changes the method of `discussion-multi-llm-bench`: its control arm (single against seats with no advisors) can no longer run, and every arm's pair config must use `discussion_advisors` as `{name, model}` and `discussion_lenses`.
- An in-flight run at deploy keeps showing `stuck` for its bounces recorded before the change (D4).
- The Burler role rename (`burler-review-chair` to `burler-review`) changes the keys `tools/tokencount` groups by, so a report spanning the change shows both names.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - advisors-startable-from-local-layers-unbounded (class decision, severity MEDIUM, open in rounds 1, 2)
  - arbiter-reader-cannot-draw-unreadable-attempt (class consistency, severity MEDIUM, open in rounds 1, 2)
  - attempt-without-verdict-reads-running-forever (class decision, severity MEDIUM, open in rounds 1, 2)
  - d11-advisors-migration-cross-layer (class design, severity MEDIUM, open in rounds 2)
  - goto-entry-has-no-round (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - card17-single-llm-comment-sites-missed (class scope, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- A frame for a run part-way through Discussion-Review shows each recipe row once: finished rows with their log lines under them, the Discussion-Review block with rounds 1..n in run order, and every later row as a bare `[ ]` line.
- Within a review segment, a history of bouncer, review, fix, bouncer, review, fix, bouncer-done renders as round 1 (bouncer, review, fix), round 2 (bouncer, review, fix) and a closing bouncer line.
- A hand-on review entry renders inside the round its output names, opening that round only when no earlier entry has, so a hand-on after an opened round adds no round heading and a restart's hand-on over an unpersisted accepted review opens its round.
- A round-3 escalation followed by an arbiter `resume` renders the awaiting line and the arbiter line inside round 3, with round 4 after them.
- A goto into a segment's review row after round k that runs a review renders the goto line at the end of round k and round k+1 opening with that review visit and no Bouncer line; a `PR-Review` history renders PR-Gate and PR-Rework as D1 rows; an unreadable recipe renders the broken line and the visits in history order.
- An `awaiting` entry renders `escalated`, and an arbiter attempt whose stop key names history length k renders right after entry k with its envelope's verdict; an attempt whose envelope carries a hand-on with a `died` cause renders `hand-on (died)`; an attempt with no envelope renders `running` before its `Deadline` and `no verdict` after it; an attempt whose k exceeds the history renders at the end; an unreadable attempt beside a readable one renders `arbiter attempt unreadable (<directory name>)` at the end of the frame, before the state line, the readable one still renders in place, and the frame still draws; a run with no arbiter directory renders no arbiter line.
- A Stuck entry with `routed_to` renders `bounced`; one without renders `stuck`; Shed writes `routed_to` on the Stuck entry of every `OnStuck` routing, and the bounce-budget count is unchanged for a history with and without it.
- A visit spanning a resumed pause entry of its slug, and a hub entry, counts its span less the paused part, overlapping entries counted once; an open visit under a live pause stops growing; an unreadable pause record counts nothing as paused.
- A one-seat table's strand role is the prefix alone; a table with a named second seat forms `prefix-name`; `Validate` refuses a named lead, an unnamed or duplicate later seat, a numeric seat name such as `2`, an invalid formed role, and a multi-seat table whose lead stencil lacks `seat-directive-chair`.
- For each converted row (Discussion-Write with no advisors and no lenses, Discussion-Write with lenses, Plan-Write with and without lenses, Darn-Write, Describe, PR-Rework), the lead seat's spec carries the same role, model, effort, version, output files, gate, interactivity, skills, segment, timeout and fork flag the row's single-producer spec held before the change; Discussion-Write's interactivity comes from the same setting under both, and only Discussion-Write's prompt changes, by the seat directive and the advisors section.
- A Plan-Write, Darn-Write, Describe or PR-Rework run started by `SingleLLMProducer` and still live is attached by the seat producer after the deploy, not respawned.
- A one-seat table's run types no line into the lead and never defers the lead's held turn end.
- Plan-Write's fresh path appends the rotation's amendment to the lead's prompt, and its resume path neither runs the rotation nor amends.
- Discussion with advisors and lenses forks from the lead only: no advisor seat sets `ForkSubagents` or carries the twill directive.
- A `discussion_advisors` entry with a blank, duplicate or invalid name, or a `discussion_lenses` or `plan_lenses` name with no fan, is refused naming the way forward.
- `MigrateConfig` maps each old `discussion_producer`, `plan_producer` and `discussion_advisors` combination as D11 lists, reports each rewrite, and leaves a migrated file unchanged on a second pass; a sparse worktree file that holds a string `discussion_advisors` list and no `discussion_producer` is refused at load naming the way forward, not emptied; a `discussion_advisors` key in a local layer or `run_settings` is refused as a guard key; a worktree file of `discussion_producer: twill` over a hub default of `seats` with advisors loads with no advisors; a worktree file of `discussion_producer: seats` with no list of its own gets the lower-layer report line.
- A `loom.yaml` that sets none of `discussion_advisors`, `discussion_lenses` and `plan_lenses` loads with no advisors and no lens forks.
- No production file names `SingleLLMProducer`, `RoleChair` or `AdvisorName` after the change.

## Notes for the plan writer

- `internal/loomcli/checklist.go` is pure over `renderChecklist`; the arbiter and pause inputs are best added as further arguments read by `checklistPoll`, keeping the renderer testable without files.
- `report-timing` plans per-phase minutes from the same history with pauses excluded; a pause-net visit-time function that report can reuse is worth placing where both reach it.
- `shedengine/activity.go` already receives `routedTo` from the one routing call site; `appendOrFold` decides folding.
- `internal/loomengine/discussion.go`'s `discussionTwillDirective` and Plan's twill spawn point are the twill sources to move into table values.
- `internal/loomshed/planwrite.go` holds the rotation that returns the amendment.
- Shuttle's attach probe matches a live run by its output files, not its role, which the attach criterion relies on.
