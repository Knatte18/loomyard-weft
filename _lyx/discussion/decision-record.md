# Decision record — batcher-v2

## Goal

Make the cost batchifier's model measured instead of guessed.
A fork's start context becomes Merriam's context at the moment the fork is spawned, which grows with every finished batch, instead of the constant `startup_context`.
`tools/tokencount -calibrate` becomes the analysis half of a learning loop: it fits the start-context coefficients and reports the cost and quality of each profile from finished runs.
`tools/tokencount` names the weft repository through one flag.
The plan spec and the plan writer's and reviewer's stencils state that a plan runs strictly in card order.
The plan gate refuses a plan whose card `**Verify:**` value, plan-level verify section or fenced code block orders a command naming the weft repo's sibling path or a fabric-driving spelling, and a sibling path in an inline code span too, and `accept-audit --batch` no longer discards a committed batch whose only fabric-reference findings are commands built from a closed list of readers.

## Scope

In:

- `internal/batcher`: the start-context model (`master_base`, `batch_growth`), the cost batchifier's dynamic program over batch positions, the breakdown fields, the `cautious` template coefficients, `doc.go`.
- The batchifier's callers in `internal/websterengine` (first init, `validate`, rebaseline), so the position of the first batch they hand over is known.
- `tools/tokencount`: one `-weft` flag; calibration of the start context per fork, peak estimate per fork with its position, a per-batch cost table and a per-profile cost and quality summary.
- `contracts/specs/loom-plan-spec.md`, `contracts/stencils/loom/loom-template-plan.md`, `contracts/stencils/loom/loom-rubric-plan-review.md`: the card-order rule.
- `internal/planparser`: the `card-fabric-reference` check; `contracts/specs/loom-plan-spec.md`: its check row; `contracts/stencils/loom/loom-template-plan.md`: the committed-data steer.
- `internal/websterengine` and `internal/webstercli`: `accept-audit --batch` for a committed batch, with the fabric-reference matching rule it shares with the plan check; `contracts/specs/refusal-spec.md`: the changed rows.

Out:

- Any change to which batches run concurrently, to webster's loop, or to the existing Plan-Gate checks (`uses-later-target`, `delete-before-reference` already refuse backward dependencies).
- Writing fitted values into `batcher.yaml` automatically; the operator edits the profile.
- A new batchifier objective: fewest batches, then the lowest largest peak, stays.
- Any change to what the implementer audit flags or how it classifies a fabric reference: it still fails the batch on every one (D14 changes only the accept route), and exposing its location-free patterns to the plan check, as D13 describes, keeps what they match.
- The imported issues' other options: giving a fabric-reference finding a path, making `recover-batch` offer `accept-audit` in more cases, and collapsing a repeated halt into one escalation.
  D13 to D15 replace the need for them, and the board section does not ask for them.
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
Bound: `budget` limits only segments of two or more cards.
A card whose own peak at its position exceeds `budget` still runs alone, and the start grows without a cap, so on a long plan this happens more often at the tail, down to every card in its own batch.
Nothing halts or warns; the batch's recorded `Estimate` and position in state.json show it, and the operator answers it by lowering `max_cards` or raising `budget` in the profile.

Rationale: an exact program keeps the batchifier a pure, testable function, as card-batching settled.
The extra dimension costs O(cards² × max_cards) segment evaluations, negligible for plans of tens of cards.

### D5 — The position of a handed-over card run is told to the batchifier

Decision: the batchifier is told how many batches run before the first card it is handed.
First init and `validate` tell it 0;
a rebaseline that keeps already-recorded batches tells it the number of batches it keeps ahead of the regrouped cards.
The partition stays computed once per run and replaced only by a rebaseline (`PATTERN-batcher-registry`, `webster-spec.md`).

Rationale: a rebaselined tail runs after the kept batches, so its forks start where Merriam's context then is.
The identity batchifier ignores the position.

Bound: the count is derived in Go from the recorded partition and state, never typed by an agent.
It is told as if Merriam were never resumed, so after a resume it is too high and the estimate conservative: more splits, never a multi-card batch whose estimate exceeds `budget`.
It is too low when Merriam did unmodelled work between batches, such as a recovery; then a batch can exceed `budget` by that work, and only the budget's headroom absorbs it.
Nothing re-checks the position at run time.

### D6 — The breakdown records the position terms

Decision: each batch's `Breakdown` records `master_base`, `batch_growth` and the batch's position alongside the other weights and components, so state.json in the weft history keeps everything the estimate used.
A breakdown recorded before this task (carrying `startup_context`) stays readable by webster and by calibration.

Rationale: the board's log step: a finished run must show its profile, coefficients and per-card components without recomputation.

### D7 — `cautious` template coefficients from this measurement

Decision: the template's `cautious` profile replaces `startup_context: 60000` with `master_base: 52000` and `batch_growth: 7000`.
All other coefficients, `budget: 450000` and `max_cards: 6` stay.
`doc.go`'s fit paragraph records where the two numbers come from (three `cautious` runs, twelve forks with the start of each measured, one of them in a resumed Merriam session) and that the in-fork coefficients were fitted under the old constant start and await a re-fit.

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
The residual spread is the 75th over the 25th percentile of the forks' measured-over-estimated start, as the existing fit reports its ratios.
Position counts per Merriam session, because a resumed Merriam starts again near its base (measured on `test-tiers`' second session: 51.4K).
A Merriam session is one webster-role session transcript, and its forks are the fork transcripts under that session that name cards; the weft records are not consulted for it.
Bound: the fit is a report and nothing consumes it.
With fewer than two distinct positions it is marked not fitted, with the reason, and the fork rows still print;
a fitted coefficient that comes out negative is shown and marked unusable, since the loader refuses a negative weight.

Rationale: the fork's first message is a direct measurement of Merriam's context at spawn, so these two coefficients can be fitted from data instead of tuned by hand.
The partition still assumes no resume, which makes a resumed run's estimate conservative.

### D10 — Calibration estimates every fork, one card or many

Decision: the peak estimate is recomputed with the named profile's weights for every fork whose cards are known, one-card and multi-card alike, as `PeakContext` of the fork's cards at the fork's position over the run's base tree.
The report sets it beside the measured peak, as now, and adds the fork's own growth: measured peak minus measured start against estimated peak minus estimated start, with the median ratio and spread per run and overall.
The reasons a run or card is left out stay, except "ran in a multi-card fork", which is retired.

Rationale: separating the start from the in-fork growth shows which half of the model is off.
Multi-card forks are now most of the data, and the estimator prices a segment as well as a card.

### D11 — Calibration reports cost and quality per profile

Decision: `-calibrate` adds a per-batch cost table (run, batch, profile, cards, measured peak, weight) and a per-profile summary: runs, batches, cards, weight per card, failed batches, recoveries, and Webster-Review findings per card.
A batch's profile is the one its `PartitionBatch` records in the run's last webster `state.json` in the weft history;
a rebaseline keeps the begun batches' profile and gives the tail the then-active one, so one run can contribute batches to several profiles.
A run with no recorded partition counts as `identity`.
Failed batches and recoveries come from that state's batch records, attributed per batch, and cards are counted from the recorded partition, or from the run's plan commit for a run with no recorded partition.
A run whose weft history holds webster records with more than one run guid was restarted with `--fresh`; the last state shows only its final attempt, while the earlier attempts' forks have no record in it, so the run's figures cannot be paired.
Such a run is listed in a marked row with that reason and enters no profile's summary line.
Webster-Review findings come from the run's round review files under `_lyx/reviews/webster/` in the weft history at its last webster record, read only through `internal/burlerengine`'s `ParseReview`, the sole reader of that format; generations a PR rework archived under `prior-generation/` are not counted.
They are a per-run figure divided by the run's card count and cannot be split by batch.
Per-batch figures (batches, cards, weight, failed batches, recoveries) go to each batch's own profile, so a run whose batches name several profiles contributes to each of them;
its findings per card appear only in its own row, which lists its profiles, and enter no profile's summary line.
A run contributes findings when its batches name one profile, it was not restarted with `--fresh` and its review files were read.
A profile line's findings per card is the findings over the cards of only the runs that contribute findings, and the line states how many runs those are;
a line with no contributing run shows the figure as not available, never zero.
A run whose state or review files cannot be read keeps its row with the missing column marked and a reason, never a silent zero.
A run whose last webster state cannot be read has no partition to name a profile, so it enters no profile line and is listed with the reason.
A weft history with no round review files marks the run's findings not read, never zero, and the run contributes none.

Rationale: the board's decide-on-data step: the operator compares profiles on weight per card without a rise in failed batches, recoveries or review findings.
The tool reports; it never edits `batcher.yaml`.

### D12 — Plans run in card order, stated where plans are written and reviewed

Decision: `loom-plan-spec.md`'s "Plan vs. schedule" section is rewritten: a valid plan is executable strictly in card-number order, a card never depends on anything a later card does, and Webster runs the cards in that order as written and never sorts them; the "DAG of intent, not itself an execution order" wording goes.
The plan template tells the writer the rule where it describes cards and the Card Index.
The plan-review rubric names a card-order violation of the spec's rule as a finding and points at the spec section; it lists no cases.

Rationale: the rule (operator, 2026-10-06) is refused by Plan-Gate since `bugfix-webster` (`uses-later-target`, `delete-before-reference`) and asserted by webster's `CheckBatchOrder` over batches the batchifier keeps in card order since card-batching, but the spec still told the writer the opposite, so Plan-Write bounced on plans Webster used to reorder.
The two Plan-Gate checks cover the ref-matched cases only; the rubric's finding covers the rest of the rule.

### D13 — The plan gate refuses a card command that reaches the weft repo

Decision: a new Plan-Gate check, `card-fabric-reference`, refuses a plan whose card text orders a command that names the weft repo.
It scans each card's `**Verify:**` value, the overview's `## verify:` section, and the fenced code blocks and inline code spans of every plan file, cards and overview alike; prose outside code is never scanned.
It flags a weft-sibling path (a name ending in the weft suffix after a separator, or a bare word ending in it outside quotes) anywhere in the scanned text, and a fabric-driving spelling (`lyx fabric`, `lyx weft`, `lyx warp` in command position) in the Verify value, the verify section and fenced blocks only.
The two differ in spans because a spelling is anchored to command position, so a span that documents a command opens with it and would be refused as an order, while a path has no such anchor and a span naming one is as likely an argument as a quotation.
The finding names the card, or the overview section for a hit there, and the matched text, and its way forward is to take the data from committed test data an earlier card creates, or to drop the command.
It runs inside the plan-format validation both plan gate sites and `validate-plan` already share.
The check and the implementer audit share one matching rule for the weft path and the spelling, not two copies: the audit's own patterns (the separator-prefixed name, the bare word, the spelling).
The audit also keeps its exact-path branch, which needs the worktree's location the plan check does not have;
the shared rule is the three location-free patterns with their preprocessing, and the audit's matches stay unchanged.
Heredoc bodies are dropped first, for all three patterns.
The separator-prefixed path pattern runs on that text with its quotes intact, so a quoted path still matches.
The bare-word and spelling patterns run on a copy with quoted spans blanked, which is what gives a bare word "outside quotes" its meaning.
The rule is declared once on the fabric side, which alone may name the weft suffix, and `planparser` receives a matcher through a narrow seam from the callers of the plan-format validation, as `websterengine` receives its matcher, so `planparser` imports neither the fabric side nor `weftname`.
The audit's patterns already live on the fabric side, so the change is a matcher that needs no worktree location.
The seam exposes the path rule and the spelling rule separately, since the check applies them to different spans, and returns the matched text for the finding, taken from the copy the pattern ran on.
The check runs the matcher once per span, so quote state never carries from one span into the next, and it scans every fenced block whatever its info string.
It reads the plan files' raw text inside `planparser`, by a model addition or a re-read, and `planparser` stays the sole parser.
Bound: the check can only refuse, and it removes no guard.
It misses what the audit's own matcher misses (a spelling hidden in `bash -c "…"` or built from a variable), the rest of a span after an unterminated quote for the bare-word and spelling patterns, a spelling in an inline code span or in prose, and any weft path not spelled as a name ending in the weft suffix;
an absolute path or `..` escape is refused only when it carries such a name.
The implementer audit stays the guard for those.

Rationale: in card-batching a card ordered `go run ./tools/tokencount … -history ../card-batching-weft`, the fork obeyed, and the audit then failed the batch three times with no route short of a full reset (#424).
Refusing at Plan-Gate catches it before any fork runs.
Only the weft path and the spelling are refused because those are what the audit flags; a read of another worktree's files by a plain relative path is the audit's concern and the steer in D15.

### D14 — `accept-audit --batch` accepts a committed batch whose fabric references are read-only

Decision: `lyx webster accept-audit --batch NN` also accepts a failed batch that has commits, when every Uncheckable entry is a pathless fabric-reference finding and the command each entry records is read-only.
The other evidence stays: the batch recorded a start commit, that commit is an ancestor of HEAD, and the worktree is clean apart from the run's own state.
A command is read-only when its recorded text is built only from the readers `cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq` and `cut`, joined by pipes, `;` or `&&`, with no output redirection of any kind, no command or process substitution and no backgrounding.
The list is closed: `sort` and `uniq` are not on it because `sort -o`, `sort --compress-program` and `uniq INPUT OUTPUT` write or run a program, and any command not named, `tee`, `xargs`, `find`, an interpreter, `lyx`, `git` and `go` included, makes the entry not read-only.
The classification is one function beside the fabric-reference matcher, and the implementer audit does not consult it: the finding still fails the batch as correctness.
Each accepted entry is recorded as a batch audit warning, as today, and the route stays the explicit `accept-audit --batch` call.
Bound: one entry that is not read-only, or that records no command text (an older record), refuses the whole call with the existing `reset --to start` then `run --fresh` steps.
The evidence shows that the recorded start still lies in HEAD's history, that nothing is uncommitted and that no listed reader can write; it does not inspect the batch's commits, and any other uncheckable entry still refuses the call.
The fabric repo's own state it cannot show, and the caller vouches for it by running the verb, as before.
Every other class, a finding with a path, and a batch that is not terminal failed behave as today.

Rationale: in shuttle-turns a fork only read other worktrees' event files, but because the batch had committed, `accept-audit --batch` refused and four cards were redone (#432).
A command that cannot write cannot have rewritten run state, so the reset protects nothing there.
The audit stays the classifier of record; widening only the explicit accept route keeps a regex's read-or-write judgement from silently passing a batch.

### D15 — The plan template steers corpus and fixture cards to committed data

Decision: `loom-template-plan.md` tells the writer that a card needing a corpus or fixture takes it from committed test data in the tree (created by an earlier card where it does not exist yet), never from live files of other worktrees or the weft repo.
It points at the `card-fabric-reference` check by its ID for what is refused and does not restate the check's rule.

Rationale: shuttle-turns' fork built its corpus from other worktrees' live event files because the card named no route, and the gate in D13 cannot see a plain relative path.

## Constraints

- `PATTERN-batcher-registry`: the partition is the active profile's, recorded in `state.json` at first init and replaced only by a rebaseline; D5 keeps that.
- `PATTERN-config-strictness`: `batcher.Active` keeps its loader; the retired `startup_context` is a strict load error (D1).
- `PATTERN-plan-generation` and `PATTERN-sole-parsers`: calibration reads plans through `planparser`, and the new plan check lives in `planparser`, over its model and the plan files' raw text, which the model does not hold for fenced blocks and inline spans;
  `PATTERN-sole-parsers` names no review format, so calibration reads Webster-Review round files through `burlerengine.ParseReview` and parses nothing itself.
- `PATTERN-refusal-way-forward`: the `card-fabric-reference` finding names its way forward and its row lands in `loom-plan-spec.md` in the same commit; the changed `accept-audit --batch` and audit rows land in `refusal-spec.md` in the same commit; a test reaches each.
- `PATTERN-gate-self-check-parity`: `card-fabric-reference` is part of the plan-format validation that Plan-Write's and Plan-Burler's gates and `validate-plan` already share, so no new verb.
- `PATTERN-no-denied-recovery`: the way forwards in D13 and D14 name no denied command; D14's steps stay `git reset --keep`, `reset --to start` and `run --fresh`.
- The weft-path and spelling match, preprocessing included, is one rule shared by the plan check and the implementer audit (D13); neither keeps its own copy, `planparser` takes no import of an engine package or of `weftname` for it, and the audit's matches do not change.
- `PATTERN-fabric-git` and `PATTERN-gitrepo-client-boundary`: `tools/tokencount` reads the weft history as it does today, through `gitrepo`'s read side; it runs no raw git.
- `PATTERN-cwd-resolution`: the weft default comes from `lyxcwd` and `fabricengine.WeftWorktree`, as now.
- `PATTERN-fabric-vocabulary`: the bare tokens weft and warp stay on the fabric side.
  Its enforcing test scans `planparser`, every test file anywhere (`tools/tokencount`'s included), `contracts/` and the docs, as a case-insensitive substring match over identifiers, string literals and comments, so a literal `-weft` flag name or a path ending in it fails too; non-test `.go` under `tools/` is not scanned.
  Only a closed exempt list passes, and of the spellings D13 refuses it holds only `lyx weft sync`, `lyx warp checkout` and `lyx.exe weft push`.
  Every changed comment, string and identifier in non-test `.go` under `internal/` outside the fabric side, and every changed `.md` or `.yaml` under `contracts/`, `docs/`, `plugins/`, `crucible/`, `cmd/` and `internal/`, says "fabric repo" or carries an exempt spelling;
  that covers the check's finding, `loom-plan-spec.md`, `refusal-spec.md`, the plan template and the `doc.go` text the acceptance criteria name.
  Tests outside the fabric side drive the check through the matcher seam, build weft-named paths with an exported helper already in `vocabExempt` (`WeftWorktree`, as the existing audit tests do), take a spelling case from `lyx fabric …` or one of the three exempt spellings, and take `tools/tokencount`'s flag name from a non-test constant.
  A committed fixture under a `testdata` directory is not scanned.
- `PATTERN-producer-pointer-rule`: the plan template and rubric point at the spec's rule, not a paraphrase of it.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`: batchifier tests are pure untagged tests; tokencount tests use committed fixtures, the integration test extends the existing one.
- `PATTERN-comment-line-breaks` and the repo's semantic line breaks for every changed comment and `.md`.
- No card command of this task's own plan may read the weft repository or another worktree's files, or run `tools/tokencount` against them: D13's check does not exist yet when this plan is gated, and the implementer audit fails such a batch (issues #424 and #432).
  The template's numbers come from this record's measurement.
- `PATTERN-documentation-lifecycle`: the design lands in `internal/batcher/doc.go` and in `tools/tokencount`'s package comment, in the same commits as the code.

## Auto-mode assumptions

- No operator answered questions; every decision here is an auto-pick.
- D7's coefficients come from three runs and twelve forks; they are a first measurement, and the learning loop is how they improve.
- D13's scan scope, D14's read-only command list and D15's steer are the discussion's reading of the board section's three wants, chosen without the operator.
- Webster-Review findings are counted per run, divided by its cards; per-card attribution of a review finding is not attempted.

## Open risks

- The in-fork coefficients were fitted under the 60000 constant start; with a lower early start and higher late start the median ratio may drift until the operator re-fits from D10's report.
- The measured sample is small and from runs of 3–5 batches; a long plan's late growth is extrapolated.
- A Merriam that does unusual work between batches (a recovery, a gate fix) grows faster than `batch_growth`; the budget's headroom absorbs it, nothing models it.
- On a long plan the start alone approaches `budget` and cards run one per batch over it (D4); only the profile's `max_cards` and `budget` and the operator's reading of the calibration report change that.
- D13 misses a spelling the matcher cannot see or one written in an inline code span or prose, and refuses a card that documents a weft path in an inline code span; the author rephrases outside code, and the implementer audit stays the guard.
- D13 scans a fenced block that carries file content as if it were commands, so a line of it opening with a spelling, or any weft path in it, is refused; the author rewrites the block or moves the text out of code.
- D14's closed reader list sends an unusual read-only command (`sort`, `uniq`, an interpreter one-liner, a `2>/dev/null` redirect, say) to the existing reset route.
- Older runs' webster state or review files may be missing or in an older shape; D11 reports them as missing rather than failing the report.

## Acceptance criteria

- A `cost` profile with `master_base` and `batch_growth` loads; one with `startup_context` fails naming `batcher.yaml`, the profile, `startup_context` and its replacements; a missing or negative `master_base` or `batch_growth` fails as other weights do.
- `PeakContext` of a segment at position k equals the same segment at position 1 plus `(k−1) × batch_growth`.
- The cost batchifier, on any input: keeps card order, keeps every one-card segment feasible, keeps every multi-card batch within `max_cards` and `budget` at its own position, and returns the fewest batches.
- A plan whose cards fit one group at position 1 splits further when the same cards are handed over at a later position.
- A card whose own peak at a late position exceeds `budget` is still emitted as a one-card batch, with its position and over-budget `Estimate` recorded.
- A rebaseline's regrouped tail is estimated from the position after the batches it keeps.
- Each recorded batch's breakdown carries `master_base`, `batch_growth` and its position; a state.json with an old breakdown carrying `startup_context` still loads.
- The template's `cautious` profile carries `master_base: 52000` and `batch_growth: 7000` and no `startup_context`.
- `tools/tokencount` accepts `-weft` alone for `-calibrate`, rejects `-history`, and reads tags, plans and webster records from the one repository.
- The calibration section lists each fork's measured start, position and estimated start, and the least-squares fit of `master_base` and `batch_growth`; a resumed Merriam session restarts its forks' positions at 1.
- With forks at fewer than two distinct positions the fit is marked not fitted with its reason, and a negative fitted coefficient is marked unusable.
- Multi-card forks get an estimate and a ratio; the in-fork growth ratio is reported per run and overall.
- The per-profile summary shows weight per card, failed batches, recoveries and Webster-Review findings per card, with runs without a recorded partition counted as `identity`, a run with an unreadable state marked with a reason and left out of every profile line, a run with no review files marked not read for findings, a run restarted with `--fresh` marked and left out of every profile line, and a run whose batches name several profiles counted per batch in each profile's line, with its findings per card in its own row only, and a profile line's findings per card taken over the runs that contribute findings alone, the line stating how many those are, and shown as not available when none does.
- `loom-plan-spec.md` states the card-order rule and no longer calls the card list "not itself an execution order"; the plan template and plan-review rubric point at it.
- A plan whose card orders a command naming a weft-sibling path in its Verify value, the verify section, a fenced block or an inline code span fails with `card-fabric-reference` naming the card, or the overview section, and the matched text; a `lyx fabric` spelling fails in the first three and not in an inline span or in prose; the same plan without them passes.
  `validate-plan` reports the same finding as the gate.
- `accept-audit --batch NN` on a failed batch with commits, whose start commit is an ancestor of HEAD and whose worktree is clean, clears entries that are all pathless fabric references from read-only commands and records one audit warning per entry;
  one entry with a redirection or an unlisted command (`tee`, `sort`, `uniq`, say), or with no recorded command text, refuses the whole call with the `reset --to start` then `run --fresh` steps;
  a start commit that is not an ancestor of HEAD and a dirty worktree refuse as before.
  The implementer audit still fails the batch on the same commands.
- `loom-template-plan.md` tells the writer to take a corpus or fixture from committed test data and points at `card-fabric-reference` by ID.
- `refusal-spec.md` rows for the fabric-reference audit and `accept-audit --batch: not acceptable` describe the committed-batch route, and `loom-plan-spec.md` has the `card-fabric-reference` row, in the commit that changes the behavior.
- `internal/batcher/doc.go` and `tools/tokencount`'s package comment describe the new model, flags and report sections; `internal/websterengine/doc.go` and `internal/planparser`'s check list describe D13 and D14.
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
- Webster-Review rounds live under `_lyx/reviews/webster/` in the weft history.
- A fabric-reference Uncheckable entry holds the matched command quoted in its detail text; D14 classifies from that.
