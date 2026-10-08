# Lens fans for Discussion and Plan review, with per-round usage

Discussion-Review and Plan-Review can now run a burler cluster review.
One reviewer session explores the artifact once, then fans out to lens forks written for a design record or a plan, so one round covers the subject broadly.
The judge narrows the fan from round 2 on, so later rounds cost less.
Every review round, solo or fanned, records its tokens and wall time, so fanned rounds can be measured against solo rounds before the fan becomes the template default.
The template default stays solo.

## What changes

- `loom.yaml` gains `discussion_fan` and `plan_fan`, each naming a fan from `burler.yaml`; empty (the default) runs the segment solo.
  The loom wiring (`internal/loomcli/wiring.go`) threads each value to both of the segment's rows, the BurlerRound row as its cluster fan and its Bouncer row as `ClusterExcludes`, so the two agree by construction.
  Config load refuses an unknown fan name, naming the key, the name and the fans that exist.
- `loom.yaml` gains a run-wide `fan_review` model (default `sonnet[high]`, same grammar as `review`).
  A fanned round's reviewer runs on it, and so do its forks, since forks always run on the parent session's model.
  The fixer keeps the segment's fix model; a solo round keeps the segment's review model.
- The embedded `burler.yaml` template gains the `discussion` and `plan` fans and their lenses (`goal-scope`, `pattern-fit`, `scenarios`, `leanness`, `decision-coverage`, `card-structure`, `verify-tests`).
  Lens texts name the rubric item they emphasize and never paraphrase its rule, so the rubric stays the authority.
- `burler.yaml` resolution falls back, per fan or lens name, to the embedded template when the operator's file lacks that name.
  Hubs seeded before this change keep a `burler.yaml` without the new fans, and seed-only reconcile never adds them, so without the fallback a hub setting `discussion_fan` would fail at round 1.
  A name the operator's file defines is used verbatim.
- Judge narrowing: round 1 runs the whole fan and drops any `exclude_lenses` a round-1 focus file carries, with a warning.
  From round 2 on, the exclusions are the union across the generation's rounds, restricted to the current fan's lenses; a lens is eligible only if the latest round ran it.
  A union that would empty the fan falls back to the exclusions held up to the previous round, and the dropped names are logged.
  A new generation starts from the whole fan.
  The judge's fanned stencil variant says exclusions are permanent for the generation.
- A fanned segment's facts file gains a lens section: the lenses run, finding counts per origin by severity and class, and the lenses already excluded.
  A solo segment's facts file is unchanged.
- Every completed review round writes `round-<N>-usage.yaml` beside its review and fixer report: fan name (empty when solo), per half the model, times, fresh and cache-read tokens, and for a fanned reviewer the fork count, lenses and the forks' share.
  An unreadable transcript records unknown tokens and never changes the round's outcome.
  Transcript reading lives in `internal/shuttleengine/claudeengine`; `shuttleengine` exposes a provider-neutral session usage on its Result.
- The declined gate-quiescence decision is re-opened.
  A gated row may authorize forks only on a BurlerRound reviewer, whose only gate is the review-parse entry.
  The ground is shuttle's wait rule: a turn end whose Stop payload reports an outstanding fork is a wait, so no gate runs while a fork is live.
  `TestNoGatedRowAuthorizesForkSubagents` is rewritten to pin exactly that, and the burler fork audit's exact fork count stays the backstop.
- Webster-Review stays solo, because its forks run no git and its subject exists only through git.
  The Webster-Burler recipe comment now says so, and a test pins that no fan reaches it.
- Docs land with the code: `internal/burlerengine/doc.go`, `internal/shedadapters/doc.go`, `internal/shuttleengine` and `claudeengine` docs, `docs/overview.md`, `contracts/specs/refusal-spec.md` and the `loom.yaml` and `burler.yaml` template comments.

## Design decisions to know

- The fan is runtime config rather than recipe: measuring fanned against solo on one deployed binary needs a switch that needs no rebuild.
  A value changed between rounds applies at the next round's wiring and can switch a segment between fanned and solo mid-generation; that changes the reviewer model but skips no gate.
- A recipe row that carries a literal `cluster-fan` and also receives an env fan fails to build, naming both sources.
- Exclusions accumulate in Go rather than relying on the judge to restate them, so narrowing is monotonic.
  The handler's holistic pass still covers an excluded lens's area every round.
- `leanness` gives the discussion fan a lens that seeks content to move or cut, so the fan does not multiply the additive bias the rubric's relocation item counters.
- Out of scope: a Webster-Review fan, flipping the template default, a CLI verb summarizing usage, and Judge token accounting.

## Check by hand

- Not run: an end-to-end loom run with `discussion_fan` and `plan_fan` set.
  The plan-level verify was not run by the implementing sessions; run `go test ./...`.
- Compare a fanned run against a solo run from the `round-<N>-usage.yaml` files before changing the template default.
  The handler's holistic pass and consolidation run on `fan_review`'s sonnet rather than the solo reviewer's opus, so weigh cross-file design sharpness against coverage.
- A Claude Code release whose Stop payload omits a live fork would let a gate run early; only the fork audit's exact count catches it, after the run.
  Fork transcript layout is a Claude Code internal; a layout change degrades usage to unknown.
- The shuttle wait rule for a gated run with an outstanding fork: confirm `TestPollEventsTick_ShellExpiry`'s row "a fork beside payload-reported shells keeps the gated run waiting" (`internal/shuttleengine/waiting_test.go`) covers it, alongside the wait tests this change adds in `internal/shuttleengine/wait_test.go`.
- Both fan-related design points in the discussion lens set, additive bias in the discussion fan and exclusion eligibility of a lens the latest round did not run, were fixed without a fresh read; read the `discussion` fan's `leanness` lens and the eligibility drop in `internal/shedadapters/focus.go` once.
- Edits outside the planned files, to check: `cmd/lyx/testdata/test-budget.yaml` (shuttleengine budget), `internal/shuttleengine/waitshadow_test.go` (zero `EndedAt` before comparing), `internal/testkit/envkit/envkit.go`, and the `shedadapters` and `shedrecipe` files that carry the fan to the Bouncer rows.
