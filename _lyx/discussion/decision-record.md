## Goal

Discussion-Review and Plan-Review can run a burler cluster review: one reviewer session explores the artifact once, then fans out to lens forks written for a design record or a plan, so one round covers the subject broadly.
The segment's judge narrows the fan from round 2 on by excluding lenses whose area is settled, so later rounds cost less.
Every review round records its tokens and wall time, so fanned rounds can be measured against solo rounds before the fan becomes the template default.
The declined gate-quiescence decision that blocked the fan is re-opened on the ground `shuttle-state` built, and its guard test is rewritten to match.

## Scope

In:

- A per-segment fan choice in `loom.yaml` for Discussion-Review and Plan-Review, solo by default.
- A run-wide model for a fanned round's reviewer session, which is the forks' model.
- Two new fans, `discussion` and `plan`, with their own lenses, in the embedded `burler.yaml` template.
- `burler.yaml` resolution that falls back, per fan or lens name, to the embedded template when the operator's file lacks that name.
- Judge narrowing: the facts file shows the judge each lens's findings, and exclusions accumulate across a segment generation's rounds.
- A per-round usage record (tokens and wall time per half, forks included) for every review segment's rounds, solo and fanned.
- Rewriting `TestNoGatedRowAuthorizesForkSubagents` and correcting the Webster-Burler recipe comment ("Single reviewer, matching both shipped segments").
- Docs in the same commit: `internal/burlerengine/doc.go`, `internal/shedadapters/doc.go`, `internal/loomengine` config docs, the `loom.yaml` and `burler.yaml` template comments.

Out:

- A fan for Webster-Review: its forks may run no git and cannot reach a diff-only subject.
- Flipping the template default to fanned; the board entry orders measurement first.
- A CLI verb that summarizes usage; the per-round records are the measurement.
- Judge (Bouncer) token accounting.

## Decisions

### D1. The fan choice is runtime config in `loom.yaml`, one key per segment

Decision: `loom.yaml` gains `discussion_fan` and `plan_fan`, each naming a fan from `burler.yaml`; empty, the template default, runs the segment solo.
The loom wiring threads each segment's value to both of its rows by row name, as `RowReviewModels` already does: the BurlerRound row's profile gets it as its cluster fan, and its Bouncer row is told `ClusterExcludes` exactly when the value is non-empty.
The shipped recipe carries no literal `cluster-fan` on any row.
A row that receives a fan from the env while its recipe config also sets a literal `cluster-fan` fails to build, with an error naming both sources.
Loom config load refuses a fan name that does not resolve (D4), naming the key, the unknown name and the fans that exist.
No `webster_fan` key exists.

Rationale: measuring fanned against solo rounds on one deployed binary needs a switch that does not require a rebuild, and one value per segment feeding both rows keeps the Bouncer and its round in agreement by construction.
Bound: the switch can only turn on a fan for the Discussion-Burler and Plan-Burler rows; a value changed between rounds takes effect at the next round's wiring and can switch a segment between fanned and solo mid-generation, which changes the reviewer model (D2) but skips no gate.

### D2. A fanned round's reviewer runs on `fan_review`

Decision: `loom.yaml` gains a run-wide `fan_review` key, a model-spec or per-round list with the same grammar and last-entry-repeats rule as `review`, template default `sonnet[high]`.
A round whose profile names a fan takes its reviewer model from `fan_review`; its fixer keeps the segment's fix model.
A solo round keeps the segment's review model (`discussion_review`/`plan_review`, else `review`).
Every entry resolves at run start, like the other model lists.

Rationale: the operator (2026-10-08) set a fanned review's forks on sonnet and a solo reviewer on opus.
Claude Code forks always run on the parent session's model, so the forks' model is the reviewer session's model; the handler's own holistic pass and consolidation run on it too.

### D3. The `discussion` and `plan` fans and their lenses

Decision: the embedded `burler.yaml` template gains these lenses and fans; every lens text is phrased as emphasis, ending with the library's "Report anything else you notice too — emphasis, never exclusion." line, and none restates or overrides the segment rubric, which stays the authority.

- `discussion: [generic, goal-scope, pattern-fit, edge-cases, acceptance]`
- `plan: [generic, decision-coverage, card-structure, verify-tests, pattern-fit]`

Lens emphases:

- `goal-scope`: the record against its board entry: goal met, scope boundaries explicit, nothing the entry asks for missing, nothing unlicensed added, no deferral or MVP phase, and whether the thing is the right one to build.
- `pattern-fit`: every decision or card against the `PATTERN.md` entries and the `pattern/` backgrounds it touches; a departure that does not name the entry it changes.
- `edge-cases`: failure, restart and resume, concurrency, empty state and invalid input; every new verb, flag, edge or weakened guard states its bound.
- `acceptance`: every decision has an acceptance criterion that would catch its regression, each criterion is observable, and must-cover scenarios name behaviors in the lowest test tier their substrate needs.
- `decision-coverage`: every Decision, Constraint and Acceptance criterion carried by some card, and no card scope the decision record does not license.
- `card-structure`: granularity, card order, member glyphs and the re-sign arrow, `ImpactSummary` conclusions, and `Custom` only as a last resort.
- `verify-tests`: the `## verify:` section runs every targeted package's tests in the tiers the rubric names, and planned tests follow `PATTERN-test-economy`.

Rationale: the shipped `standard` lenses (error-handling, test-gaps) are written for code, and the rubrics' own "Also flag" lists name what a design record and a plan most often get wrong.

### D4. `burler.yaml` resolves per name over the embedded template

Decision: fan and lens resolution reads the operator's `burler.yaml` first and falls back to the embedded template for a fan or lens name the operator's file does not define; an absent file resolves wholly from the template.
A name the operator's file defines is used verbatim, so an operator override of `discussion`, `plan` or any lens wins.
`burler.yaml` stays seed-only for reconcile, and its template header states the fallback.

Rationale: every hub seeded before this task has a `burler.yaml` without the new fans, and seed-only reconcile never adds them, so without a fallback a hub that sets `discussion_fan` would fail at round 1.
Bound: the fallback only supplies names absent from the operator's file; a standard fan the operator deleted resolves again from the template, which changes nothing unless a profile or `loom.yaml` names it.

### D5. Judge narrowing accumulates across a generation's rounds

Decision: round 1 runs the configured fan whole and ignores any `exclude_lenses` the seed call writes.
From round 2 on, the lenses excluded from round N are the union of `exclude_lenses` across the segment generation's focus files for rounds 2 to N.
When that union would leave no lens, round N runs round N-1's lens set and the new exclusions are dropped with a warning naming them.
An archived or fresh generation starts from the whole fan again.
The judge stencil's fanned variant (`focusSchemaMarkers`) tells the judge that an exclusion is permanent for the generation, to exclude a lens only when its area is settled (the latest round's findings from it carry nothing at MEDIUM or worse and its area is not in the next round's `focus`), and never to restate earlier exclusions.

Rationale: the board asks the judge to narrow the fan each bounce; a union held in Go makes narrowing monotonic without relying on the judge to restate every earlier exclusion, and the handler's holistic pass still covers an excluded lens's area every round.
Bound: the judge can drop any lens, but never the last one and never in round 1; an excluded lens returns only with a new generation.

### D6. The judge sees each lens's findings

Decision: for a segment told `ClusterExcludes`, Go's round facts file gains a lens section: the lenses the latest round ran, the count of its findings per `origin` (each `lens:<name>` and `handler`) by severity and class, and the lenses already excluded for the next round.
A solo segment's facts file is unchanged.

Rationale: the judge reads the facts, the latest review and the previous ledger, never the fan; without the lens list and per-lens findings it cannot tell which lens names exist or which area is settled.

### D7. Gate quiescence with forks is re-opened and settled by shuttle's wait

Decision: a gated row may authorize fork subagents only on a BurlerRound row, where the forking half is the reviewer and its only gate is the review-parse entry (`ReviewGateEntry`); writer rows (DiscussionWrite, PlanWrite, Describe, PRRework) still never fork, and the fixer never forks.
`TestNoGatedRowAuthorizesForkSubagents` is rewritten to pin exactly that over the recipe and the wiring's env-supplied fans, with its failure message naming the new ground.
The ground is shuttle's existing wait rule from `shuttle-state`: a turn end whose Stop payload reports an outstanding fork is a wait, never a finished turn, so no gate runs and no output check finishes the run while a fork is live; a shuttle wait test pins that rule, extended if no existing test covers a gated run with an outstanding fork.
The burler fork audit (`auditClusterRound`, exact fork count) stays the backstop.

Rationale: the decision was declined because turn-idle alone would no longer mean the agent is finished; `shuttle-state` made a turn end with outstanding forks a wait, which is that missing signal.
Bound: forks are allowed on BurlerRound reviewers only, bounded by the round's `review_timeout_min`; a Stop payload that omits a live fork is caught by the fork audit's exact count, not by the gate.

### D8. Every review round records its usage

Decision: after each round, Go writes `round-<N>-usage.yaml` beside the round's review and fixer report in the segment's run directory, for every review segment, solo and fanned.
Per half it records the model, start, end and wall time, fresh tokens (input plus cache creation plus output) and cache-read tokens; the reviewer's totals include its forks, with the fork count, the lenses run and the forks' share broken out.
The round's record also names the fan (empty when solo).
Reading usage is a provider capability in `claudeengine`, beside the fork audit's transcript knowledge; an unreadable transcript records that half's tokens as unknown and never fails or delays the round.
The record is durable: it lives in the run directory the status seam commits.

Rationale: the board requires tokens and wall time per round and per segment against solo rounds before the fan becomes the default; per-round records from both arms make that a reading, and a segment's figures are the sum of its rounds.

### D9. Webster-Review stays solo, and its recipe comment says why

Decision: the Webster-Burler comment ending "Single reviewer, matching both shipped segments" is corrected to state that Webster-Review stays solo because forks run no git and its subject exists only through git, while Discussion-Review and Plan-Review take their fan from `loom.yaml`.
A recipe test pins that Webster-Burler carries no `cluster-fan` and that the wiring supplies it no fan.

Rationale: the old comment's premise (both other segments solo) is false once D1 lands.

## Constraints

- `PATTERN-review-round`: a fanned round keeps review-before-fix; consolidation is part of the review, and the fixer starts only after Go accepts the consolidated review.
- `PATTERN-shuttle-provider-seam`: transcript and fork-transcript usage reading lives in `internal/shuttleengine/claudeengine`; `shuttleengine` and `burlerengine` see a provider-neutral reading.
- `PATTERN-config-strictness`: `loomengine` loads strictly; the new keys get template defaults and validation at load.
  `burler.yaml` keeps its direct-read loader; the per-name template fallback is resolution, not a degrade of a missing file to `LoadOrTemplate`.
- `PATTERN-durable-vs-ephemeral-state`: the usage record is durable and lives in the review run directory under `_lyx`.
- `PATTERN-sole-parsers`: focus files are read only through the existing `readRoundFocus`; no new parser for discussion or plan files.
- `PATTERN-stencil-ownership` and the stencils' conditional-free rule: the judge's fanned-variant text stays Go-held in `focusSchemaMarkers`.
- `PATTERN-refusal-way-forward` applies in spirit to the config-load refusal of an unknown fan name: it names the key and the fans that exist.
- `PATTERN-test-speed` and `PATTERN-test-economy`: new coverage is untagged and extends existing tests where one already covers the surface; no new `llm`-tier test.
- `PATTERN-documentation-lifecycle`: the built design lands in `doc.go` files in the same commit.
- Fork discipline is unchanged: unnamed `subagent_type: "fork"` only, read-only, no git, no nested Agent calls, enforced by the PreToolUse hook and the fork audit.

## Auto-mode assumptions

- The fan defaults to off in the template until the measurement in D8 has been read; the default flip is a later one-line template change, as the board entry orders.
- One run-wide `fan_review` key is enough; per-segment fan models are not asked for.
- `sonnet[high]` is the fan model default, matching `webster_review`.
- The lens names and fan compositions in D3 are this record's pick; five forks per fan matches the shipped `standard` fan's size.
- Exclusions are permanent within a generation (D5) rather than re-decided by the judge each round.
- No usage summary verb: per-round records suffice for a one-off comparison.

## Open risks

- The handler's holistic review and consolidation run on `fan_review`'s sonnet, not the solo reviewer's opus; a fanned round may judge cross-file design less sharply than an opus solo round, which the measurement should weigh against coverage.
- A Claude Code release whose Stop payload omits a live fork would let a gate run early; the fork audit's exact count catches the shortfall only after the run is done.
- Fork transcript layout is a Claude Code internal; a layout change degrades usage to unknown, and the fork audit with it.

## Acceptance criteria

- With `discussion_fan` and `plan_fan` set, Discussion-Burler and Plan-Burler rounds run with that fan and their Bouncers ask for `exclude_lenses`; with them empty, the rounds run solo and the Bouncers' prompts name no `exclude_lenses`.
- Webster-Burler never receives a fan, from the recipe or from the wiring.
- A recipe row carrying a literal `cluster-fan` that also receives an env fan fails to build, naming both.
- Loom config load refuses an unknown fan name in `discussion_fan` or `plan_fan`, naming the key and the existing fans, and refuses an invalid `fan_review` entry.
- A fanned round's reviewer runs on the `fan_review` entry for its round and its fixer on the segment's fix model; a solo round's reviewer runs on the segment's review model.
- Fan resolution: an operator file without `discussion` resolves it from the template; an operator-defined `discussion` wins; an absent file resolves from the template; an undefined name in both is an error naming the fan.
- The template's `discussion` and `plan` fans resolve, and every lens they name exists.
- Exclusion: round 1 runs the whole fan despite seed excludes; round N excludes the union of rounds 2..N; a union that would empty the fan keeps round N-1's set and logs the dropped names; a new generation runs the whole fan.
- A fanned segment's facts file lists the lenses run, per-origin finding counts by severity and class, and the excluded lenses; a solo segment's facts file is unchanged.
- The rewritten quiescence test fails for a gated writer row that forks and passes for a gated BurlerRound row with a fan; shuttle's wait keeps a gated run waiting at a turn end that reports an outstanding fork.
- Every completed review round, solo and fanned, leaves `round-<N>-usage.yaml` with per-half model, wall time, fresh and cache-read tokens, and for a fanned reviewer the fork count, lenses and fork share; an unreadable transcript records unknown tokens and the round's outcome is unchanged.
- `go test ./...` passes.

## Notes for the plan writer

- Recipe and its guards: `contracts/recipes/loom-recipe.yaml`, `internal/loomrecipe/gatequiescence_test.go`, `internal/loomrecipe/clusterexcludes_test.go`.
- Row wiring: `internal/loomcli/wiring.go` (`RowReviewModels` is the pattern for per-row env values), `internal/shedrecipe/entries_burler.go`, `internal/shedrecipe/entries_bouncer.go`, `internal/shedrecipe/recipe.go`.
- Models and config: `internal/loomengine/config.go`, `internal/loomengine/review.go`, `internal/loomengine/template.yaml`.
- Fans: `internal/burlerengine/config.go` (`LoadConfig`, `ResolveFan`), `internal/burlerengine/template.yaml`, `internal/burlerengine/profile.go` (`applyClusterExclude`).
- Narrowing and facts: `internal/shedadapters/burler.go` (per-round `ClusterExclude`), `internal/shedadapters/focus.go`, `internal/shedadapters/bouncerfacts.go`, `internal/shedadapters/bouncerprompt.go`.
- Fork waits and audit: `internal/shuttleengine/wait.go`, `internal/shuttleengine/claudeengine` (fork audit and `usage.go`), `internal/burlerengine/cluster.go`.
