# Decision record: per-role-models

## Goal

Every loom role that today shares its model with another role gets a `loom.yaml` key of its own: each Bouncer row's judge pass, each Bouncer row's seed pass, and PR-Rework.
Go stops trusting the judge alone on convergence: it refuses a CONVERGED verdict while a BLOCKING finding is open, and refuses a ledger that drops a key the previous round's ledger carried.
Judge and seed passes are recorded in the per-round usage records beside the review and fix halves, so their cost is read from the run's own files instead of session transcripts.

## Scope

In:

- `loom.yaml` keys `seed`, `discussion_judge`, `plan_judge`, `webster_judge`, `discussion_seed`, `plan_seed`, `webster_seed` and `rework`, their resolution, and the threading of the resolved values into each Bouncer row's seed and judge spawns and into PR-Rework's spawn.
- Retiring the Bouncer row's recipe-level `model`, `effort` and `version` config keys.
- The Go-side convergence guard in the Bouncer's harvest of a judge pass.
- `seed` and `judge` entries in `round-<N>-usage.yaml`.
- The docs that describe these: `internal/loomengine/template.yaml` key comments, `contracts/specs/llm-model-spec.md`'s roles section, the `internal/shedadapters` and `internal/loomengine` package docs where they state the judge model, the verdict contract or the usage record, the loom recipe's Bouncer-row comments, and a row in `contracts/specs/refusal-spec.md`'s shed section.

Out (operator rulings on the board entry):

- Webster's implementer and verify-fix forks, the Bouncer escalation fork and the fan's lens forks: a fork runs on its parent session's model.
- `landing.yaml` `describe` and `darn.yaml` `writer`: hub-wide only, no per-worktree key wanted.
- A cold-start implementer with its own model: the separate entry `weaver`.
- Any change to which model a role runs on by default: with every new key empty or absent, every role runs on exactly the model it runs on today.
- Any Haiku move for the judge or the seed: the operator ruled on 2026-10-11 that the Bouncer's Haiku work is dropped.

## Decisions

### D1. Per-segment judge keys over the run-wide `judge`

Decision: `loom.yaml` gains `discussion_judge`, `plan_judge` and `webster_judge`, each one model-spec string.
Empty, the template default, takes the run-wide `judge` key; a set value replaces it for that segment's Bouncer row only.
`judge` keeps its name, meaning and default `sonnet[medium]`.
Rationale: it is the shape `discussion_review`/`plan_review`/`webster_review` already use over `review`, so an operator reads one convention across the file, and an existing `loom.yaml` behaves exactly as before.

### D2. A seed model separate from the judge's, run-wide and per segment

Decision: `loom.yaml` gains a run-wide `seed` key and per-segment `discussion_seed`, `plan_seed` and `webster_seed`, each one model-spec string, all with template default empty.
A segment's seed model resolves in this order: its per-segment seed key when set, else `seed` when set, else that segment's resolved judge model (D1).
So with every new key empty or absent, each seed pass runs on exactly the model it runs on today, and a set seed key moves the seed off the judge's model without touching the judge.
Rationale: the board asks for a seed key separate from the judge's on every row; the per-segment layer keeps the seed and judge keys symmetric, and the judge fallback keeps today's behavior, since this task adds keys and changes no role's default model.

### D4. Judge and seed settings are resolved per segment at run start

Decision: the loom wiring resolves the judge and seed model-spec of each review segment (Discussion-Review, Plan-Review, Webster-Review) once at run start, through the model registry, and threads them onto the recipe environment keyed by segment.
The Bouncer entry looks up its row's `segment` and fills separate seed and judge model, effort and version fields on the Bouncer's config.
A Bouncer row whose segment has no resolved settings is a wiring error at recipe build, not a runtime fallback.
Every per-segment key is parsed at config load and resolved at run start, so a bad alias fails before round 1, as `ResolveReview` does for the review lists.
Rationale: resolution before the first spawn is the existing contract for every loom model key, and keying by segment matches how the review and fix lists are already threaded.

### D5. The Bouncer row's recipe-level `model`, `effort` and `version` keys are retired

Decision: the Bouncer recipe entry no longer accepts `model`, `effort` or `version`; a recipe naming one is rejected as an unknown key, as any other unknown key is.
`loom.yaml` is the only source of a Bouncer pass's model.
Rationale: one recipe `model` key cannot name two passes, no Bouncer row in any embedded recipe sets it, and a recipe-literal model that silently outranks config is the exact problem the board entry names.

### D6. PR-Rework gets `rework`

Decision: `loom.yaml` gains `rework`, one model-spec string; empty, the template default, takes `plan`.
PR-Rework keeps `plan_timeout_min` as its timeout; no `rework_timeout_min` key is added.
Rationale: empty-takes-`plan` keeps every run's behavior unchanged until an operator sets the key, and the board asks for a model key only.

### D7. Go refuses a CONVERGED verdict with an open BLOCKING finding

Decision: when the Bouncer harvests a judge pass that this Call spawned or attached to and the verdict is CONVERGED, Go refuses the judgment if either holds:

- the round's ledger written by that judge carries an entry with `status: open` and `severity: BLOCKING`;
- the round's review report carries a finding of severity BLOCKING, or does not parse through the review parser the facts file already uses.

The second check follows the judge's own decision rule: any BLOCKING finding in the latest round, or a parse-error row for it, rules CONTINUE, and the judge stencil forbids relabelling a BLOCKING finding downward.
Bound: the guard only ever turns a CONVERGED into a re-judge; it never accepts anything the judge did not, and CONTINUE and CIRCLING verdicts pass it untouched.
A verdict already on disk at Call entry, settled by an earlier Call, is not re-checked.
Rationale: a weaker or careless judge can otherwise approve past a BLOCKING finding with nothing in Go to catch it.

### D8. Go refuses a ledger that drops a previous key

Decision: on harvesting a judge pass of round N ≥ 2, for any verdict, Go refuses the judgment when a key in round N-1's ledger is absent from round N's ledger.
The check runs only when round N-1's ledger parses; an unreadable or malformed previous ledger skips it with a logged warning, since there is nothing trustworthy to compare against, and the facts file already shows that ledger as a parse-error row.
Bound: the check compares keys only, not status, class or severity; it enforces the judge stencil's existing lossless carry-forward rule and adds no new obligation on the judge.
Rationale: the decision record's carried findings come only from the judge's ledger, so a dropped key loses a finding silently.

### D9. A refused judgment is archived and re-judged

Decision: a judgment refused by D7 or D8 takes the route a retired verdict word already takes: Go archives the round's three judge outputs (verdict, ledger, focus), so the round reads as unjudged, and returns a degraded Stuck whose Reason names the refusal cause and ends in `way forward: the round is re-judged`.
The round producer's unjudged-round check hands straight back and the Bouncer's next Call re-judges.
Bound: each refusal spends one bounce unit, so a judge that keeps producing refused judgments reaches the segment's `review_max_bounces` budget escalation to the parent; the guard cannot loop.
The refusal is bound by `PATTERN-refusal-way-forward`: it lands with a row in `contracts/specs/refusal-spec.md`'s shed section, whose way forward is the transition the run takes by itself (the re-judge), and a test that reaches it.
Rationale: overriding the verdict to CONTINUE in Go would leave the next round with the CONVERGED pass's empty focus file and a ledger Go has just judged untrustworthy; re-judging keeps the judge the only writer of its three files.

### D10. Judge and seed passes go into `round-<N>-usage.yaml`

Decision: the usage record gains two optional lists, `seed` and `judge`, each entry one pass in the existing per-half shape: model, effort, start, end, wall seconds and the token reading (`tokens_known`, fresh and cache-read tokens).
A judge pass appends to round N's `judge` list; a seed pass appends to round 1's `seed` list; a re-judged round therefore shows every pass it took.
An entry is recorded for every pass whose shuttle call returned a result, with tokens unknown when the provider could not read the session; a pass that fails to spawn records nothing.
Each writer reads the record if present and replaces or appends only its own part: the Burler round writer replaces `review`, `fix`, `fan` and `lenses` and keeps `seed` and `judge`; the Bouncer appends to its list and keeps the rest.
A malformed existing record is not merged into: the writer archives it the way a stale judge output is archived and writes a fresh record holding only its own part, with a logged warning.
A failed usage write only logs, as the Burler round's does today, and never changes the Bouncer's outcome.
The seed entry of a new generation lands in that generation's round-1 record, never in one the clear-and-re-seed archived.
Rationale: the board asks for judge and seed in the usage files like other roles; one record per round keeps a round's whole cost in one file, and the Bouncer and Burler rows of one segment never run at once, so read-and-replace has a single writer at a time.

### D11. Readers of the usage record ignore a record without a review half

Decision: the lens-fact and lens-exclusion readers treat a record that has no review half (a seed-only or judge-only record, left when the Burler round's own write failed) as a missing record.
Rationale: today a failed Burler write leaves no file and those readers report it missing; a Bouncer-created record must not turn that into a record claiming a solo round with no lenses.

## Constraints

- `PATTERN-refusal-way-forward`: D9's refusal names its way forward, gets its shed-section row in the same commit, and a test reaches it.
- `PATTERN-config-strictness`: loom config keeps its present load mode; new keys are template keys that gap-fill like every other.
- `PATTERN-shuttle-provider-seam`: the usage reading comes from shuttle's provider-neutral session reading; no provider specifics enter `shedadapters` or `loomengine`.
- `PATTERN-shed-recipe-registry`: the Bouncer constructor stays reached through `Lookup`; only its accepted config keys and its environment fields change.
- `PATTERN-told-geometry`: the Bouncer keeps writing usage under the run directory it is handed and derives no path.
- `PATTERN-sole-parsers`: review reports are not among the sole-parser files; D7 reads them through the same review parser the facts file already uses, adding no second parser.
- `PATTERN-test-economy` and `PATTERN-test-speed`: guard and usage behavior are pinned at the Bouncer's public producer surface with fakes, extending the existing Bouncer and round-usage tests where a table fits.
- `PATTERN-documentation-lifecycle`: the design lands in the package docs named in Scope, not in a design doc.
- Docs-in-the-same-commit rule (CLAUDE.md): the template comments, the model-spec roles section, the package docs and the refusal row land with the code.
- Test budget: a card whose new top-level tests push a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- A1. The board's "each Bouncer row judge and seed pass" means a per-segment key for both passes, layered over run-wide keys, rather than per-segment judge keys and one run-wide seed key (D1, D2).
- A3. Per-segment judge and seed keys take one model-spec each, not a per-round list as the review and fix keys do; no per-round need is stated.
- A4. `rework` defaults to empty and falls back to `plan` (D6) rather than copying `plan`'s current value into the template.
- A5. D7's review-side check uses the latest round's review report only, not the fixer report: the judge reads no fixer report and its decision rule counts the review's findings.
- A6. A refused judgment carries no feedback into the re-judge prompt; the bounce budget bounds repeats (D9).
- A7. The usage record keeps its file name and existing fields; `seed` and `judge` are added beside them (D10).

## Open risks

- A Sonnet judge that rules CONVERGED with a BLOCKING finding in the review, which the stencil already forbids, now spends a bounce unit per refusal; if that happens often, escalations to the parent rise until the stencil or the judge model changes.
- The usage record's merge-on-write relies on the Bouncer and Burler rows of one segment never running concurrently; a future producer that runs them in parallel must revisit D10.

## Acceptance criteria

- With every new key empty or absent, a run spawns each Bouncer's judge on `judge`, each seed on its segment's judge model, and PR-Rework on `plan`, so every role runs on the model it runs on today.
- Setting `seed` moves every segment's seed to it, and setting `plan_judge` alone moves Plan-Bouncer's seed with its judge while no seed key is set.
- Setting `plan_judge` changes only Plan-Bouncer's judge model; Discussion-Bouncer's and Webster-Bouncer's judges stay on `judge`, and a set `seed` or `plan_seed` keeps Plan-Bouncer's seed off it.
- Setting `webster_seed` changes only Webster-Bouncer's seed model, and the judge of that row is unaffected.
- Setting `rework` changes PR-Rework's model, and Plan-Write keeps `plan`.
- A malformed or unknown-alias value in any new key fails the run before round 1 with an error naming the key.
- A Bouncer row in a recipe that names `model`, `effort` or `version` is rejected at recipe build.
- A judge CONVERGED with an open BLOCKING ledger entry is archived and re-judged, spending one bounce unit, and the Reason names the cause and the way forward.
- A judge CONVERGED over a review report that carries a BLOCKING finding, or one that does not parse, is archived and re-judged.
- A judge of round 2 or later whose ledger lacks a key from round N-1's ledger is archived and re-judged, whatever its verdict.
- A malformed previous ledger skips the key check without refusing the judgment.
- A CONVERGED verdict with no open BLOCKING entry, no BLOCKING review finding and a lossless ledger settles exactly as today.
- Repeated refusals reach the segment's budget escalation and never loop.
- After a round is judged, its `round-<N>-usage.yaml` holds the review and fix halves and one `judge` entry per judge pass, with model, effort, run times and token reading.
- Round 1's record holds a `seed` entry after the seed pass, and the Burler round's later write keeps it.
- A judge or seed pass whose provider could not read the session records `tokens_known: false` and no token fields.
- A record holding only a seed or judge part is read as missing by the lens-fact and lens-exclusion readers.
- `contracts/specs/refusal-spec.md` has the shed row for D9's refusal, and `contracts/specs/llm-model-spec.md` names the new roles.

## Notes for the plan writer

- Judge resolution: `internal/loomengine/review.go` `ResolveJudge`; the review and fix per-segment pattern to mirror is `ResolveReview`/`resolveSegmentModels` in the same file, with keys declared in `internal/loomengine/config.go` and the template in `internal/loomengine/template.yaml`.
- Threading: `internal/loomcli/wiring.go` builds the recipe environment; `internal/shedrecipe/recipe.go` declares `JudgeModel`/`JudgeEffort`/`JudgeVersion`; `internal/shedrecipe/entries_bouncer.go` reads the row's `model`/`effort`/`version` and falls back to them.
- The Bouncer's seed and judge spawn specs both read `b.cfg.Model` in `internal/shedadapters/bouncer.go`.
- Retired-verdict precedent for D9: `retireLegacyVerdict` in `internal/shedadapters/bouncer.go`; ledger and verdict parsing in `bouncerfiles.go`; severity constants in `bouncerfacts.go`.
- Usage record: `internal/shedadapters/roundusage.go`; readers in `bouncerfacts.go` (`computeLensFacts`) and `focus.go` (`lensesRunUnderFan`).
- PR-Rework: `internal/loomengine/rework.go` `ReworkSpec`.
- The token report in `tools/tokencount` reads transcripts by session role and needs no change.
