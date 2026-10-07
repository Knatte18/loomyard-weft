# Decision record — batcher-v2

## Goal

Make the cost batchifier's model measured instead of guessed.
A fork's start context becomes Merriam's context at the moment the fork is spawned, which grows with every finished batch, instead of the constant `startup_context`.
`tools/tokencount -calibrate` becomes the analysis half of a learning loop: it fits the start-context coefficients and reports the cost and quality of each profile from finished runs.
`tools/tokencount` names the weft repository through one flag.
The plan spec and the plan writer's and reviewer's stencils state that a plan runs strictly in card order.

## Scope

In:

- `internal/batcher`: the start-context model (`master_base`, `batch_growth`), the cost batchifier's dynamic program over batch positions, the breakdown fields, the `cautious` template coefficients, `doc.go`.
- The batchifier's callers in `internal/websterengine` (first init, `validate`, rebaseline), so the position of the first batch they hand over is known.
- `tools/tokencount`: one `-weft` flag; calibration of the start context per fork, peak estimate per fork with its position, a per-batch cost table and a per-profile cost and quality summary.
- `contracts/specs/loom-plan-spec.md`, `contracts/stencils/loom/loom-template-plan.md`, `contracts/stencils/loom/loom-rubric-plan-review.md`: the card-order rule.

Out:

- Any change to which batches run concurrently, to webster's loop, or to the Plan-Gate checks themselves (`uses-later-target`, `delete-before-reference` already refuse backward dependencies).
- Writing fitted values into `batcher.yaml` automatically; the operator edits the profile.
- A new batchifier objective: fewest batches, then the lowest largest peak, stays.
- The fabric-reference audit (`plan-fabric-reference` owns it).
- Prompt-cache TTL (`cache-ttl-per-role` owns it).

## Decisions

### D1 — A fork starts at Merriam's base plus a growth per batch before it

Decision: the weight `startup_context` is replaced by two coefficients, `master_base` (Merriam's context when it spawns its first fork) and `batch_growth` (the context Merriam gains per finished batch).
The batch at position k (1-based, counted over the whole run's partition) starts at `master_base + (k−1) × batch_growth`;
`fork_messages × message_context` and the read and write terms are added as today.
A profile still carrying `startup_context` fails to load with an error naming `batcher.yaml`, the profile, the retired key and its two replacements, as `alone_above` does.

Rationale: an implementer fork is an in-session fork of Merriam, so its first message carries Merriam's whole context.
Measured on the three `cautious` runs (see Notes), the fork's first-message context equals Merriam's last context before the spawn, starting at 50–54K and growing 2.5–13K per batch.
A constant is too high early and too low late, which is where a large late batch can overrun the budget.

### D2 — `master_base` is a fitted constant, not a computation over Merriam's files

Decision: `master_base` is a coefficient fitted from finished runs.
lyx does not compute it from the sizes of the stencil, `CLAUDE.md`, PATTERN or the plan.

Rationale: the board entry expected the base to be computable exactly before the run, but it is not.
Merriam starts at about 20.6K (Claude Code's system prompt and tools, `CLAUDE.md`, the Webster stencil) and then orients itself, reading `00-overview.md` and as much of the codebase as it chooses, until it reaches 50–54K at the first fork.
That second part is an agent's choice, not a file list, and lyx has no tokenizer to price files exactly.
The measured spread across runs is about 6%, small against the budget, so a fitted constant is as good as a computed sum and costs no new machinery.

### D3 — Growth counts batches, not cards

Decision: growth is `batch_growth` per batch run before; there is no per-card growth coefficient.

Rationale: what comes back to Merriam between forks is one report digest and the output of `begin-batch`, `record-batch` and `await-batch` per batch.
Over the eight measured growth steps the growth did not track the previous batch's card count (6 cards gave +11.9K and +6.6K, 1 card gave +6.4K, 4 cards +2.9K), so a per-card term would fit noise.

### D4 — The cost batchifier's dynamic program tracks the batch position

Decision: a segment's peak now depends on its position, so the exact dynamic program runs over (number of batches so far, last card placed).
It keeps its objective and tie-breaks: the fewest batches, then the lowest largest peak, then the last batch shortest.
Feasibility is unchanged: a one-card segment is always feasible, and a segment of two or more cards needs at most `max_cards` cards and a peak within `budget`.
Since the start grows with position, a card that fits a group early can run alone late; no position makes the split infeasible.

Rationale: an exact program keeps the batchifier a pure, testable function, as card-batching settled.
The extra dimension costs O(cards² × max_cards) segment evaluations, negligible for plans of tens of cards.

### D5 — The position of a handed-over card run is told to the batchifier

Decision: the batchifier is told how many batches run before the first card it is handed.
First init and `validate` tell it 0;
a rebaseline that keeps already-recorded batches tells it the number of batches it keeps ahead of the regrouped cards.
The partition stays computed once per run and replaced only by a rebaseline (`PATTERN-batcher-registry`, `webster-spec.md`).

Rationale: a rebaselined tail runs after the kept batches, so its forks start where Merriam's context then is.
The identity batchifier ignores the position.

### D6 — The breakdown records the position terms

Decision: each batch's `Breakdown` records `master_base`, `batch_growth` and the batch's position alongside the other weights and components, so state.json in the weft history keeps everything the estimate used.
A breakdown recorded before this task (carrying `startup_context`) stays readable by webster and by calibration.

Rationale: the board's log step: a finished run must show its profile, coefficients and per-card components without recomputation.

### D7 — `cautious` template coefficients from this measurement

Decision: the template's `cautious` profile replaces `startup_context: 60000` with `master_base: 52000` and `batch_growth: 7000`.
All other coefficients, `budget: 450000` and `max_cards: 6` stay.
`doc.go`'s fit paragraph records where the two numbers come from (three `cautious` runs, eleven forks, the start of each measured) and that the in-fork coefficients were fitted under the old constant start and await a re-fit.

Rationale: 52K is the mean measured start of a first fork (50.3K–53.5K); 7K the mean measured growth per batch (2.5K–13.0K).
The budget leaves room for the spread.

### D8 — One `-weft` flag in `tools/tokencount`

Decision: `-history` is removed.
`-weft` names the prime's weft repository for every read the tool makes: the archive tags of finished runs, the runs' plan commits and the runs' webster records.
Its default stays the current worktree's weft sibling, so `-calibrate <profile>` needs no other flag.

Rationale: both flags named the same repository; one flag with one default removes a required argument and a way to point the two at different repositories.

### D9 — Calibration fits the start context per fork

Decision: for every Webster fork of every counted run, `-calibrate` reports its measured start context (the context of its first counted message), its position (the forks before it in the same Merriam session, plus one) and the start the profile estimates for that position.
It fits `master_base` and `batch_growth` by least squares of measured start over position − 1 across all forks, and reports the fitted pair and its residual spread beside the profile's values.
Position counts per Merriam session, because a resumed Merriam starts again near its base (measured on `test-tiers`' second session: 51.4K).

Rationale: the fork's first message is a direct measurement of Merriam's context at spawn, so these two coefficients can be fitted from data instead of tuned by hand.
The partition still assumes no resume, which makes a resumed run's estimate conservative.

### D10 — Calibration estimates every fork, one card or many

Decision: the peak estimate is recomputed with the named profile's weights for every fork whose cards are known, one-card and multi-card alike, as `PeakContext` of the fork's cards at the fork's position over the run's base tree.
The report sets it beside the measured peak, as now, and adds the fork's own growth: measured peak minus measured start against estimated peak minus estimated start, with the median ratio and spread per run and overall.
The reasons a run or card is left out stay, except "ran in a multi-card fork", which is retired.

Rationale: separating the start from the in-fork growth shows which half of the model is off.
Multi-card forks are now most of the data, and the estimator prices a segment as well as a card.

### D11 — Calibration reports cost and quality per profile

Decision: `-calibrate` adds a per-batch cost table (run, profile, cards, measured peak, weight) and a per-profile summary: runs, batches, cards, weight per card, failed batches, recoveries, and Webster-Review findings per card.
A run's profile is the one its recorded partition names, read from the run's last webster `state.json` in the weft history;
a run with no recorded partition counts as `identity`.
Failed batches and recoveries come from that state's batch records.
Webster-Review findings come from the run's Webster-Review round files in the weft history, read only through the package that owns their format (`PATTERN-sole-parsers` where it applies), and are divided by the run's card count.
A run whose state or review files cannot be read keeps its row with the missing column marked and a reason, never a silent zero.

Rationale: the board's decide-on-data step: the operator compares profiles on weight per card without a rise in failed batches, recoveries or review findings.
The tool reports; it never edits `batcher.yaml`.

### D12 — Plans run in card order, stated where plans are written and reviewed

Decision: `loom-plan-spec.md`'s "Plan vs. schedule" section is rewritten: a valid plan is executable strictly in card-number order, a card never depends on anything a later card does, and Webster runs the cards in that order as written and never sorts them; the "DAG of intent, not itself an execution order" wording goes.
The plan template tells the writer the rule where it describes cards and the Card Index.
The plan-review rubric names a card that depends on a later card (uses a symbol a later card creates, or references what an earlier card deletes) as a finding, pointing at the Plan-Gate checks that refuse it.

Rationale: the rule (operator, 2026-10-06) is enforced by Plan-Gate since `bugfix-webster` and by the batchifier's contiguous segments since card-batching, but the spec still told the writer the opposite, so Plan-Write bounced on plans Webster used to reorder.

## Constraints

- `PATTERN-batcher-registry`: the partition is the active profile's, recorded in `state.json` at first init and replaced only by a rebaseline; D5 keeps that.
- `PATTERN-config-strictness`: `batcher.Active` keeps its loader; the retired `startup_context` is a strict load error (D1).
- `PATTERN-plan-generation` and `PATTERN-sole-parsers`: calibration reads plans through `planparser` and review files only through their owning parser.
- `PATTERN-fabric-git` and `PATTERN-gitrepo-client-boundary`: `tools/tokencount` reads the weft history as it does today, through `gitrepo`'s read side; it runs no raw git.
- `PATTERN-cwd-resolution`: the weft default comes from `lyxcwd` and `fabricengine.WeftWorktree`, as now.
- `PATTERN-producer-pointer-rule`: the plan template and rubric point at the spec's rule, not a paraphrase of it.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`: batchifier tests are pure untagged tests; tokencount tests use committed fixtures, the integration test extends the existing one.
- `PATTERN-comment-line-breaks` and the repo's semantic line breaks for every changed comment and `.md`.
- No card command may read the weft repository or another worktree's files, or run `tools/tokencount` against them (`plan-fabric-reference`, issues #424 and #432: the implementer audit fails such a batch).
  The template's numbers come from this record's measurement.
- `PATTERN-documentation-lifecycle`: the design lands in `internal/batcher/doc.go` and in `tools/tokencount`'s package comment, in the same commits as the code.

## Auto-mode assumptions

- No operator answered questions; every pick in the support log's Question ledger is an auto-pick.
- D2 departs from the board entry's "base computable exactly before the run", on the measured evidence that Merriam's orientation is agent-driven and its spread small.
- D3 drops the board's "per-card" growth option on eight measured steps.
- D7's coefficients come from three runs and eleven forks; they are a first measurement, and the learning loop is how they improve.
- Webster-Review findings are counted per run, divided by its cards; per-card attribution of a review finding is not attempted.

## Open risks

- The in-fork coefficients were fitted under the 60000 constant start; with a lower early start and higher late start the median ratio may drift until the operator re-fits from D10's report.
- The measured sample is small and from runs of 3–5 batches; a long plan's late growth is extrapolated.
- A Merriam that does unusual work between batches (a recovery, a gate fix) grows faster than `batch_growth`; the budget's headroom absorbs it, nothing models it.
- Older runs' webster state or review files may be missing or in an older shape; D11 reports them as missing rather than failing the report.

## Acceptance criteria

- A `cost` profile with `master_base` and `batch_growth` loads; one with `startup_context` fails naming `batcher.yaml`, the profile, `startup_context` and its replacements; a missing or negative `master_base` or `batch_growth` fails as other weights do.
- `PeakContext` of a segment at position k equals the same segment at position 1 plus `(k−1) × batch_growth`.
- The cost batchifier, on any input: keeps card order, keeps every one-card segment feasible, keeps every multi-card batch within `max_cards` and `budget` at its own position, and returns the fewest batches.
- A plan whose cards fit one group at position 1 splits further when the same cards are handed over at a later position.
- A rebaseline's regrouped tail is estimated from the position after the batches it keeps.
- Each recorded batch's breakdown carries `master_base`, `batch_growth` and its position; a state.json with an old breakdown carrying `startup_context` still loads.
- The template's `cautious` profile carries `master_base: 52000` and `batch_growth: 7000` and no `startup_context`.
- `tools/tokencount` accepts `-weft` alone for `-calibrate`, rejects `-history`, and reads tags, plans and webster records from the one repository.
- The calibration section lists each fork's measured start, position and estimated start, and the least-squares fit of `master_base` and `batch_growth`; a resumed Merriam session restarts its forks' positions at 1.
- Multi-card forks get an estimate and a ratio; the in-fork growth ratio is reported per run and overall.
- The per-profile summary shows weight per card, failed batches, recoveries and Webster-Review findings per card, with runs without a recorded partition counted as `identity`, and a run with unreadable records marked with a reason.
- `loom-plan-spec.md` states the card-order rule and no longer calls the card list "not itself an execution order"; the plan template and plan-review rubric point at it.
- `internal/batcher/doc.go` and `tools/tokencount`'s package comment describe the new model, flags and report sections.
- The full suite passes.

## Notes for the plan writer

Measurement taken during Discussion (2026-10-07), read-only from the Merriam transcripts of the `cautious` runs; a fork's first-message context equals Merriam's last context before the spawn in every case:

| run | Merriam session | fork | start (K) | peak (K) | growth from previous (K) |
|---|---|---|---|---|---|
| shuttle-turns | 1 | 1 | 53.5 | 253.7 | — |
| shuttle-turns | 1 | 2 | 65.5 | 257.6 | +11.9 after 6 cards |
| shuttle-turns | 1 | 3 | 68.3 | 194.9 | +2.9 after 4 cards |
| shuttle-turns | 1 | 4 | 70.8 | 384.1 | +2.5 after 2 cards |
| test-tiers | 1 | 1 | 50.3 | 162.5 | — |
| test-tiers | 1 | 2 | 60.2 | 290.5 | +9.9 after 3 cards |
| test-tiers | 1 | 3 | 73.2 | 123.2 | +13.0 after 5 cards |
| test-tiers | 1 | 4 | 79.6 | 219.6 | +6.4 after 1 card |
| test-tiers | 2 (resume) | 1 | 51.4 | 87.8 | — |
| run-recovery | 1 | 1 | 52.1 | 198.3 | — |
| run-recovery | 1 | 2 | 58.7 | 201.8 | +6.6 after 6 cards |
| run-recovery | 1 | 3 | 61.8 | 308.1 | +3.1 after 5 cards |

Merriam's first message is about 20.6K in every session.
Fork weight per card from `tools/tokencount` (weight as the tool defines it, recoveries included): the `cautious` runs 0.22–0.44M (about 0.30M over 54 cards), the earlier identity runs `card-batching`, `bugfix-webster`, `bugfix-sessions`, `bugfix-fabric` and `run-pushes` 0.22–0.57M (about 0.40M over 52 cards).

- `PartitionBatch.Breakdown` already carries profile, weights and per-card components into state.json; D6 extends it rather than adding a record.
- Webster run records land in the weft as `loom: webster run record for <slug>` commits holding `_lyx/webster/state.json`.
- Webster-Review rounds live under `_lyx/reviews/` in the weft history.
