# Decision record — burler-split-budget

## Goal

Cut the cost of a review segment's Burler round and let a perch end on its own.
The round becomes two Shed rows, a reviewer row and then a fixer row, each a single fresh session; the fixer spawns only when the review has findings and orients only on the review and the targets it names.
Each perch gets its own bounce budget and an auto-accept flag, so a spent budget can accept the work as it stands instead of escalating to the parent.

## Scope

In:

- Splitting every `BurlerRound` row into a reviewer row and a fixer row, in `contracts/recipes/loom-recipe.yaml` (Discussion-Review, Plan-Review, Webster-Review) and `contracts/recipes/darn-recipe.yaml` (Darn-Review), since all four perches share the one engine.
- The Shed adapter, `burlerengine` and seat-table changes the split needs, and `lyx burler run` running both halves in sequence.
- Skipping the fixer session on a review with no findings.
- A lean fixer prompt, and a review format whose findings carry the change to make.
- Per-perch bounce budget and auto-accept keys in `loom.yaml` for Discussion-Review, Plan-Review and Webster-Review.
- The docs, spec and PATTERN amendments these changes need, in the same commit.

Out:

- Auto-accept for darn's Darn-Review perch: `darn.yaml` already carries its own `review_max_bounces`, and the board entry names only Discussion, Plan and Webster.
- Setting the hub's own `loom.yaml` (Webster budget 1 with auto-accept): `_lyx/config/` is untracked per-worktree config the operator sets after landing.
- Per-task run settings on the board entry (`task-run-settings`); the per-perch keys here are what that note will set.
- The lens fan's pre-read and fork budget (`fan-master-preread`) and the Burler kick-start pack (`kick-start-pack`): both now land on the review row's prompt only.
- Folding `SingleLLMProducer` into `MultiLLMProducer` (`seats-replaces-singlellm`); the Burler adapter stays a deliberate exception to that fold (D2).
- The seat hold deferral (`hold-deferral-end-on-held`): with zero advisors Burler no longer uses it, and the note stays about seats Discussion.

## Decisions

### D1. The round is two rows, sequenced by Shed

Each perch's single `BurlerRound` row becomes two rows in the same segment: `<Perch>-Burler-Review` then `<Perch>-Burler-Fix` (for example `Webster-Burler-Review`, `Darn-Burler-Fix`).
Routing: the Bouncer's `on_stuck` names the review row; the review row's `on_stuck` names the fix row; the fix row's `on_stuck` names the Bouncer.
Both rows' `on_done` name the same row as their `on_stuck`, the existing guard against an empty `on_done`; neither row ever returns Done.
The new row names get their matching `internal/loomshed` name constants, which the recipe's coverage guard pins.
Rationale: a Done by any row of a segment ends the segment's bounce episode (`shedengine`'s `episodeStuckCount`), so a Done between the halves would reset the Bouncer's budget every round; Stuck hand-offs keep the episode intact.

### D2. Each row runs a one-seat table on the seats engine

Each row runs one fresh session as the chair of a `seatengine` table with zero advisors: the reviewer on the review row, the fixer on the fix row.
Each half has its own role prefix, `burler-review` and `burler-fix`, so the two strands never share a name and a fix row's probe can never attach to a live reviewer.
The Burler-specific adapter logic (round resolution from disk, prior-round hydration, bounded retry on died or timed-out, probe and resume of a live session, archiving, BudgetExempt hand-backs) stays in the Burler adapter in `internal/shedadapters`, split per half; the rows do not use the generic `MultiLLMProducer` type.
Rationale: `MultiLLMProducer` maps a chair's success onto Done, and D1 needs Stuck hand-offs; the seats engine stays the one way an LLM session is run, and the adapter count does not grow, since `BurlerProducer` exists today.

### D3. Review row behaviour

The review row resolves the round to review from disk and spawns the reviewer with today's review orientation (rubric, fasit, target, focus file, prior rounds, cluster fan).
When the review passes its gate, Go writes the round's acceptance record, holding a digest of the accepted review, and the row hands to the fix row with a BudgetExempt Stuck.
The acceptance record is what tells "accepted review, fixer pending" from an orphan review: round resolution, the complete-round check and archiving read it, and a review on disk without one is an orphan, archived and reviewed again.
A round whose acceptance record is already on disk spawns nothing and hands on BudgetExempt, so a restart, or a pass-through of a complete-but-unjudged round, reaches the Bouncer through the fix row.
A reviewer whose gate fails after its attempts, or that dies twice, stays a hard error with the round's paths archived, as today: a Stuck with no review would be misread by the Bouncer as a seed call.

### D4. Fix row behaviour and the empty-review skip

The fix row reads the current round's acceptance record and accepted review:

- findings present: spawn the fixer, hold the hand-off on the row's gates (D5), check the review still matches the accepted digest, then hand to the Bouncer BudgetExempt;
- zero findings: the row runs but spawns no session; it writes a Go-authored fixer report saying the review held no findings, which parses like any other fixer report, and hands to the Bouncer BudgetExempt;
- fixer report already on disk: spawn nothing and hand on BudgetExempt;
- fix gate exhausted: archive both round paths and hand to the Bouncer with a counted Stuck and an empty Path, as today's gate-failed round does, so the next round reviews afresh;
- review changed after acceptance: hard error, as today;
- no acceptance record for the round (reachable only by a goto into the fix row): hard error whose way forward is a goto to the perch's review row.

Rationale: the skip keeps every downstream reader (round completeness, hydration, the judge) on the existing review and fixer-report pair, so none of them gains an empty-round special case.
Bound: the skip lets through only an accepted review with zero findings of any severity; any finding, NIT included, spawns the fixer, since every finding is fixed at every severity (`PATTERN-review-round`).
A skipped Webster round runs no verify gate, since nothing changed.
The fix row's MaxBounces caps its counted gate-failed Stucks per episode, as the single row's did.

### D5. Gates split by half

The review row carries the review-parse entry, plus D7's `fix` check, as its gate; the fix row carries the row config's gates (`discussion`, `plan` or `verify`) with the fixer-report repair the round applies today.
The recipe's `gates` key moves to the fix rows unchanged.

### D6. Lean fixer orientation

The fixer's prompt holds: the fix orchestrator, the review file, the target paths the review's findings name, the fix scope's write surface, the commit rule of `fix-scope: source`, and the dispute and fixer-report rules.
It does not hold the rubric, the fasit, the PATTERN overview, the focus file, prior rounds, the decision record or the plan cards, and the explore step is rendered for the reviewer only.
The fixer still validates each finding against the code it reads and fixes or disputes it with evidence.
Rationale: duplicate orientation is the measured waste of today's fixer, and archived rounds show it applying every finding.

### D7. The review carries the fix

Every finding in the review file gains a `fix` key: the exact change to make and where, specific enough to apply without the rubric or fasit.
The review row's gate refuses a current-round finding with an empty `fix`, so the reviewer is re-prompted until each finding carries one; parsing a prior round's review for hydration or circling evidence does not require the key.
The review format the reviewer is pointed at and the review step stencil state the key.
Rationale: the fixer no longer holds the reviewer's orientation, so the review is its whole instruction.

### D8. No reviewer-fixer channel

The seat channel between fixer and reviewer goes, with its stencil text, the ready-marker wait inside the fixer's turn and `burlerengine`'s handoff hook between seats.
The reviewer session ends when its review is accepted, and the fixer never messages it.
Rationale: sequential rows have no live reviewer to ask, D7 makes the review self-contained, and a transcript search found no fixer message to a `burler-advisor`.

### D9. `PATTERN-review-round` amended

The background file and its PATTERN line change in the same commit:

- `premise false` means false against the code the fixer reads; the fasit and the rubric's `Do not flag` list are no longer fixer dispute grounds, since the fixer holds neither: the reviewer applies both before writing, and the Bouncer's judge, which reads the rubric, rules on a finding that breaks them;
- "the fixer touches nothing before Go has accepted the review" holds by sequencing: the fix row spawns only after the review row's acceptance record is on disk.

### D10. Per-perch budget and auto-accept keys

`loom.yaml` gains, for each of `discussion`, `plan` and `webster`:

- `<perch>_max_bounces`: the perch's bounce budget; 0, the default, takes `review_max_bounces`;
- `<perch>_auto_accept`: false by default.

`review_max_bounces` and `review_circling_checkpoint` stay as the run-wide defaults; model keys keep picking each half's model per round as today.
Every row of a perch's segment carries the perch's resolved budget, and the Bouncer's bounce seam and loom's routing read the same per-perch value.
A budget of N gives N judged Burler rounds, so 1 gives exactly one.
A per-perch key below 0, or a resolved budget below 1, is refused naming the key it came from; the refusal-spec row "config: review key below 1" is amended to name the per-perch keys, with its test.
The template comments state the relation per perch: a checkpoint above a perch's budget means that perch never rules CIRCLING and reaches the budget escalation, or the accept, instead.
Defaults keep today's behaviour.

### D11. Auto-accept settles a spent budget

With `<perch>_auto_accept` true, a round whose budget is spent and that has no recorded decision is settled by the Bouncer as an accept: it records the decision as an automatic budget-cause accept, distinguishable from a parent's `lyx loom circling accept` in the circling decision file only, then takes the existing accept path (carry-over of the open findings into the decision record's `## Open risks` under the existing accepted class, approve and commit seams, Done).
No escalation record or parent notice is written.
Bound: it fires only for the budget cause; a CIRCLING verdict below the spent budget still escalates to the parent; an unreadable decision or escalation record still degrades as today; a failed or unknown bounce seam counts as not spent and falls to Shed's generic budget block, which halts for a human; a perch with the flag false behaves exactly as today.
A recorded decision settles as recorded whatever the flag's current value, and an escalation already recorded for the round stands: auto-accept never overrides an Awaiting the parent was told about.

## Constraints

- `PATTERN-review-round`: amended by D9; every other clause holds.
- `PATTERN-refusal-way-forward`: D4's missing-acceptance error and D10's amended budget refusal each name the way forward, carry their refusal-spec row and a test.
- `PATTERN-shed-recipe-registry`: any new engine name is registered in `internal/shedrecipe`'s one map.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: the split fix and review stencils are read at call time from the stencils directory, and the fixer prompt points at the review format rather than restating it.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-role-skills-typed`, `PATTERN-spawn-color`: each half's opening stencil renders both directives, and its skills and color segment ride the launch spec.
- `PATTERN-agent-name`: `burlerengine` owns the two role constants; reed forms the names.
- `PATTERN-config-strictness`: the new `loom.yaml` keys go through the existing loader and template.
- `PATTERN-test-economy`, `PATTERN-test-speed`: routing and skip scenarios are pinned with fakes at the adapter and recipe surfaces; real-engine smoke tests change only where the two-seat round they pin is gone.
- Row names are durable resume identities: an in-flight run parked on a retired `*-Burler` row is restarted, not migrated, as the recipe header already says for retired rows.
- Docs touched in the same commit: `burlerengine`'s, `burlercli`'s and `shedadapters`'s `doc.go`, both recipe headers, and the `loom.yaml` template comments.

## Auto-mode assumptions

- D2's reading of the board's "a `MultiLLMProducer` with zero advisors" as a zero-advisor seat table under the Burler adapter.
- D4's empty-review bound and D11's budget-only scope.
- D6 drops the PATTERN overview from the fixer, as the board entry asks, although lyx's spawned agents otherwise get it.
- D10's override shape, and no per-perch circling checkpoint.
- The darn split without darn auto-accept.
- The fixer stays the round's only friction-note writer, so a skipped round leaves none.

## Open risks

- A review whose `fix` instructions are vague leaves the lean fixer guessing; the judge still sees the result, and only verify checks a Webster fix after an auto-accepted single round (accepted by the operator).
- Without the PATTERN overview, a code fix may depart from a PATTERN rule that no test pins and the reviewer did not name.

## Acceptance criteria

- Each perch routes Bouncer → review row → fix row → Bouncer, and no row of a Burler pair returns Done.
- A full round with findings spawns one reviewer session and then one fixer session under distinct role names, never both at once.
- A review with zero findings spawns no fixer, writes a Go-authored fixer report that parses, runs no verify gate, and the Bouncer judges the round.
- A review with only a NIT finding spawns the fixer.
- A reviewer whose gate fails after its attempts is a hard error with the round archived and no fixer spawn.
- A current-round finding with an empty `fix` fails the review gate and re-prompts the reviewer; a prior round's review without `fix` still hydrates.
- A fix gate exhausted after its attempts archives both round paths and reaches the Bouncer with a counted Stuck and an empty Path.
- A review edited after acceptance fails the fix row.
- The fixer's prompt holds the review and the target paths and holds none of the rubric, fasit, PATTERN overview, focus file or prior rounds.
- Restart: a run killed with an acceptance record and no fixer report resumes at the fix row without re-running the reviewer; one killed with a review on disk but no acceptance record re-reviews the round; one killed with a live reviewer or fixer resumes or re-spawns that half only; one killed after the fixer report hands on to the Bouncer without a spawn.
- The Bouncer's budget counts the same judged rounds as today: with a budget of N, the Nth CONTINUE escalates or auto-accepts, and the pass-through hand-backs never consume it.
- With `webster_max_bounces: 1` and `webster_auto_accept: true`, the segment runs exactly one Burler round and a CONTINUE on it settles Done and moves to Describe.
- `<perch>_max_bounces` 0 takes `review_max_bounces`; a set value applies to that perch's rows only; a negative key or a resolved value below 1 is refused naming the key.
- With `<perch>_auto_accept` true and the budget spent, the round settles Done with the open findings carried into `## Open risks` and an automatic accept on disk, and no parent notice is written.
- With auto-accept true, a CIRCLING verdict before the budget is spent still escalates to the parent, and a failing bounce seam halts rather than accepts.
- With every new key at its default, routing, budgets and escalations match today's.
- `lyx burler run` runs the reviewer then, only when the review has findings, the fixer.

## Notes for the plan writer

- The budget rule and the BudgetExempt hand-backs are documented on `NewBouncer` and `BurlerProducer` in `internal/shedadapters`; the accept path to reuse is `settleUnconverged`, and the budget check is `budgetReached`.
- `loomrecipe.Routing` takes the single `ReviewMaxBounces` at two `loomcli` call sites, the bounce seam in `wiring.go` and `loadRouting` in `arm.go`; both, and `applyReviewBudget`, move to per-perch values.
- Today's round handoff, including the review-bytes check D4 keeps, is `decideRound` in `internal/burlerengine/handoff.go`.
