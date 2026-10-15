# Decision record — run-pane-timing

## Goal

Make the per-run status pane (the checklist `lyx loom checklist` draws for loom and darn runs) tell the operator where a run's time went: row times to the second, a total per review round, each gate's own time and the rework its refusals caused, and a fixed number per pane entry.
Make every spawned review-segment session's name say which segment it serves, so the Bouncer and Burler sessions of Discussion, Plan, Webster, Merge and Darn review can be told apart in reed's strand names, window names and bar buttons.

## Scope

In:

- The checklist frame (`internal/loomcli/checklist.go`): seconds in closed times, round totals, gate lines, rework attribution, entry numbers.
- A seconds-precision time format in `internal/statuspane`, beside `FormatMinutes`.
- Per-entry gate spans in shuttle's gate outcome, carried through the producer output into the shed history entry, Webster's verify gate included.
- One recipe-rows lookup covering every embedded recipe, batten's included.
- Row numbers in batten's prime runs overview (`internal/battencli/overview.go`), the one pane a batten run has.
- Segment-prefixed session roles for the Bouncer (seed, judge) and Burler (review, fix) sessions of every review segment in the loom and darn recipes, and every stencil and doc that names those roles.
- Each touched package's `doc.go` and the recipe file headers, in the same commit.

Out:

- The runs overview's and the orch's own time columns: they keep `FormatMinutes`.
- `lyx loom status` text and `lyx report` output; board note `report-timing` owns a timing report.
- Roles outside review segments (writers, driver, Merriam, arbiter, alignment, friction, tackler, conflict, treadle): each is held by one row or one stop at a time, and the checklist already names the row it serves.
- A standalone `lyx burler run` session: it has no recipe segment and keeps its unprefixed roles.
- Migrating history entries written before this change: they carry no gate spans and draw as they do today.

## Decisions

### D1 — Closed times to the second, open times in whole minutes

Decision: a new `statuspane` format renders a duration as `45s`, `12m05s`, `1h05m12s`, and from one day on as `FormatMinutes` does (`2d03h`).
The checklist draws every closed span with it: a closed visit line, a row total, a round total, a gate line and a segment heading whose spans are all closed.
A time that includes the open visit (the current producer's running visit, or a total that sums it) is drawn with `FormatMinutes`.
This narrows the board's item 1 for the running row only.

Rationale: `statuspane.Pane` erases the screen and the scrollback on every redraw and redraws only on a changed frame; a ticking seconds counter would redraw and wipe the operator's scrollback every second for the whole run.
Closed spans never change, so seconds there cost no redraws.

### D2 — A total per review round

Decision: each `round <n>` line of a review block reads `round <n> <time>`, the sum of the spent times of the visit lines placed in that round, gotos excluded since they carry no time.
The segment heading keeps its segment total and round count.

Rationale: the round's parts are already grouped under the round line; the sum needs no new record.

### D3 — Entry numbers fixed by the recipe

Decision: the checklist numbers its entries `1.`, `2.`, … in recipe order, after the box: `[x] 3. Discussion-Write 4m12s`.
A review segment's block (a segment holding a Bouncer row) is one entry and takes one number, at the place of its first row; its rounds and bounces carry no numbers.
A row drawn from history that is not in the recipe (`(not in recipe)`) and the `recipe unreadable` fallback carry no number.
The checklist reads rows through one lookup over every embedded recipe (loom, darn and batten), so a recipe added to that lookup is numbered with no checklist change.

Rationale: a recipe's row list is fixed, so a number names the same entry on every frame of every run of that recipe.

### D4 — Batten's numbers in the runs overview

Decision: the prime runs overview's stage cell is prefixed with the stage's entry number: `5. Land` for a batten row; while batten is in Run-Shed, the child's row as `<Run-Shed's number>.<child number> <row>`, for example `4.7 Plan-Write`, the child number being the one D3 gives that row in the child's checklist.
A stage whose number cannot be read (row not in the recipe, unreadable seed) shows the name alone.

Rationale: batten runs have no per-run checklist; the overview line is the batten run's pane, and the board entry names batten among the recipes to number.
The dotted form keeps a batten number and a child number from reading alike.

### D5 — Gate spans recorded by shuttle

Decision: shuttle's gate outcome reports, per gate entry, beside its attempt count, the spans that entry took, each a start and an end on the run clock and a kind:
a gate span for each closure evaluation, and for a pending entry such as `parent-review` the time held pending;
a rework span from each re-prompt that entry sent to the writer's next arrival.
A span still open when the run finalizes (session lost, deadline expired, a later terminal failure, a failed send) ends at finalize and counts.
The spans are kept in `Wait`'s memory like the attempt count; a re-attach after a driver restart starts them afresh, as it starts the attempt count.
The outcome carries the spans on every result, done, failed or session-lost alike.

Carriers: the producers that today copy a gate count into the history entry (`MultiLLMProducer` and `BurlerFixProducer`, through `internal/shedadapters/gateattempts.go`) copy the spans too.
The Webster row becomes a carrier: its result surfaces the Merriam run's gate outcome, verify gate included, and the adapter copies its count and spans the same way.
The shed history entry stores the spans as an optional list beside `gate_attempts`, on every entry kind that carries `gate_attempts` today, reroute entries included.
The field is additive and tolerant (PATTERN-tolerant-records): an entry without it reads as today.
Folding keeps today's test, which compares `GateAttempts` and not the spans; a folded entry appends the new entry's spans to its own.

Rationale: the records today keep only the total re-prompt count, and the findings file and run directory are deleted on finalize, so no reader can rebuild gate time after the fact.
Shuttle is the only process that sees each evaluation and each re-prompt.
Spans with start and end, not bare durations, let the checklist take paused time out of them as it does for visits.

### D6 — Gate lines in the checklist

Decision: under a visit line whose history entry carries gate spans, the checklist draws one line per gate entry with a span: `gate <name> <gate time>`, then ` refused <n> rework <rework time>` when it re-prompted.
Each time is the sum of that entry's spans of that kind, each span less the pause record's coverage within it and clamped at zero.
A visit with only the legacy `gate_attempts` keeps today's ` gate attempts <n>` suffix; an empty span list draws no gate line.
The gate name is drawn as recorded, whether or not a recipe names it.

Rationale: the in-row gates (validate-discussion, parent-review, plan validation, Webster's verify gate) live inside a row's visit, so their lines belong under that visit, and they obey the visit's pause rule.

### D7 — Rework after a gate row's refusal

Decision: a visit that ends bounced (a Stuck entry with `routed_to`) in no segment holding a Bouncer row is a gate refusal.
Its rework is the history entry right after the bounce entry when that entry is a visit of the routed-to row; otherwise the refusal has no rework visit.
The rework visit line reads ` rework for <gate row>`, and the gate row's own line adds ` rework <time>`, the sum of its rework visits.
In a segment holding a Bouncer row the bounces are not attributed this way: the review rounds are the rework and D2's round totals show it.
PR-Gate and PR-Rework sit in `PR-Review`, which holds no Bouncer row, so D7 covers them.

Rationale: a row gate such as PR-Gate sends the run back to the producer that made its artifact; the status history already records the route, so this needs no new record.

### D8 — Segment-prefixed Bouncer and Burler roles

Decision: the Bouncer, BurlerReview and BurlerFix rows take an optional recipe config key `role_prefix`, accepted by each row's config decoder and checked with `agentname.ValidateRole`.
With it set, the row's sessions carry the roles `<prefix>-bouncer-seed`, `<prefix>-bouncer-judge`, `<prefix>-burler-review` and `<prefix>-burler-fix`; with it empty they keep today's roles.
The Burler prefix reaches the seat table's role prefix through `burlerengine`'s profile, the path the color segment already takes.
The loom recipe sets `discussion`, `plan`, `webster` and `merge`, and the darn recipe `darn`, on every row of each review segment.
A recipe test pins that every Bouncer and Burler row in a segment sets the key, that the rows of one segment share one value, and that no two segments share one.
A collision of a prefixed role with another role in the worktree is left to reed's `NumberRole`, which numbers it.
The strand name is formed only by `agentname`, through reed (PATTERN-agent-name), so the strand name, the window name and the bar button all carry the prefix with no reed change.
The stencils and docs that name `bouncer-seed`, `bouncer-judge`, `burler-review` or `burler-fix` as recipe roles are updated, and any test that pins those names follows.
`tools/tokencount` keeps tallying by the recorded role, so its tallies split per segment.

Rationale: an explicit key gives short, readable names (`ly:<slug>:plan-burler-review`) and leaves standalone Burler untouched.
The spawn color stays on the launch spec's segment (PATTERN-spawn-color); the role is a separate field.
Shuttle's attach matches on output files, not the role, so a session spawned under the old role is still attached.

## Constraints

- PATTERN-tolerant-records: the new history field is optional and never refused when absent or unknown; writes go through `internal/state`.
- PATTERN-shed-producer-seam: the span type `shedengine` stores is a plain stdlib struct; `shedengine` imports nothing new.
- PATTERN-shuttle-provider-seam: spans are measured in shuttle's provider-neutral wait loop, never in `claudeengine`.
- PATTERN-agent-name: roles are passed as role strings; only `agentname` forms the full name.
- PATTERN-shed-run-directory: the overview reads the child's seed and recipe only through `shedrun`.
- PATTERN-test-speed: span tests inject the run clock and the checklist's `now`, never wait.

## Auto-mode assumptions

- "Gates" means both kinds a run has: the in-row gates a writer row runs and the gate rows that bounce to another producer. A perch's Bouncer is covered by the round totals.
- "In the pane" for item 5: the checklist already names each review row with its segment (`Discussion-Burler-Review`), so the item's gap is the session names reed shows.
- The rework span of an in-row gate runs from the re-prompt send to the next arrival, the writer's turn end that the next evaluation reads.

## Open risks

- A run in flight when this lands has history entries without gate spans; its frames show the legacy `gate attempts` suffix for those visits.
- A session live across the binary change keeps its unprefixed name until it ends.
- A gated run re-attached after a driver restart reports only the spans from the re-attach on.
- Token tallies grouped by role no longer line up with runs from before the change.

## Acceptance criteria

- A frame with closed visits shows seconds (`45s`, `12m05s`, `1h05m12s`), and a frame whose only change is the open visit's next second is byte-identical to the previous frame.
- A review block's round lines each show the sum of their lines' times; a round holding the open visit shows it in whole minutes.
- Entries are numbered in recipe order, a review segment taking one number; a history-only row and the recipe-unreadable fallback carry none; a darn run's frame numbers by the darn recipe.
- Batten's overview stage cell shows `<n>. <row>` for a batten row and `<n>.<m> <row>` for the child's row in Run-Shed; an unknown row shows the name alone.
- A gated run that passes first try reports each entry with zero attempts and its gate span; one refused twice reports two attempts and a rework span from each re-prompt to the next arrival; a pending entry reports its held span; times come from an injected clock.
- A deadline expiring after a re-prompt, and a session lost after one, end the open span at finalize, and the outcome carries the spans.
- The Webster row's history entry carries the verify gate's count and spans.
- A folded history entry carries the spans of every entry folded into it.
- A history entry with spans draws one gate line per entry; a pause inside a pending gate's span is left out of its time; a span ending before it starts draws zero; an empty list draws no gate line; an entry with only `gate_attempts` draws today's suffix; an entry with unknown extra keys still decodes.
- A gate row's bounce marks the visit right after it as its rework when that visit is of the routed-to row, and adds its time to the gate row's line; a bounce followed by a goto or another row has no rework visit; a bounce in a segment holding a Bouncer row is not attributed.
- With `role_prefix` set, the Bouncer's seed and judge and the Burler's review and fix sessions carry the prefixed roles; with it empty the roles are today's; an invalid prefix is refused when the row is built.
- The recipe test fails when a review segment's row lacks `role_prefix`, when a segment's rows disagree, or when two segments share a prefix.

## Notes for the plan writer

- Gate entries in the recipes: the `gates:` lists under the writer rows of `contracts/recipes/loom-recipe.yaml` and `darn-recipe.yaml`; Webster adds its verify gate in `internal/websterengine/runlevel.go`.
- History folding is `appendOrFold`/`foldsInto` in `internal/shedengine/run.go`; reroute entries are built in `internal/shedengine/reroute.go`.
- Role constants today: `bouncerSeedRole` and `bouncerJudgeRole` in `internal/shedadapters/bouncer.go`, `reviewRolePrefix` and `fixRolePrefix` in `internal/burlerengine`; `awaitLiveSeed`'s comment claiming Attach matches on the role is stale.
- `loomrecipe.Rows` serves loom and darn today.
- Board note `report-timing` can read the history entry's gate spans; `loom-model-key-names` can reuse the `role_prefix` values as its per-segment vocabulary; `run-verbs-shared` lists `lyx loom checklist` among verbs that serve non-loom runs.
