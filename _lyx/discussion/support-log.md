## Interview

Autonomous run: no operator answered; every question below was settled by an auto-pick recorded in the Question ledger.

1. Exploration.
   Read the board entry and its dependencies (`shuttle-state`, `review-loop-convergence`, both done) and the related note `multi-agent-producer`.
   Read the loom recipe, both recipe guard tests, the burler engine doc and config, the loom config template and model resolution, the judge stencil and its Go-held markers, the round focus plumbing, the shuttle wait rules on outstanding forks, and both review rubrics.
2. Key facts found.
   The judge-to-round `exclude_lenses` channel is built end to end but per round only, and the judge never sees lens names or per-lens findings.
   Forks inherit the parent session's model, so "forks on sonnet" means the fanned reviewer session runs on sonnet.
   `burler.yaml` is seed-only, so already-seeded hubs lack any new fan.
   Shuttle already treats a turn end reporting an outstanding fork as a wait, and a gated run with an outstanding fork keeps waiting.
   No token or wall-time accounting exists per review round.
3. Scope question: is the fan a recipe literal (the board's proposal) or runtime config?
   The board requires measurement against solo rounds before the fan becomes default, which needs both arms on one deployed binary, so runtime config won (D1).
4. Model question: how does a fan get its own model when forks cannot have one?
   A `fan_review` key sets the fanned reviewer session's model (D2).
5. Lens design: which lenses suit a design record and a plan?
   The plan lenses and the discussion fan's `leanness` come from the rubrics' "Also flag" lists, the other discussion lenses from the discussion writer's interview categories (D3).
6. Seeded hubs: how do existing `burler.yaml` files get the new fans?
   Per-name fallback to the embedded template (D4).
7. Narrowing: per round or cumulative, and how does the judge know what to drop?
   Cumulative union in Go from round 2, never emptying the fan (D5), with a lens section in the facts file (D6).
8. Blocker: gate quiescence with forks.
   Settled on shuttle's existing fork-wait rule, with the guard test rewritten rather than deleted (D7).
9. Measurement: what does "the task measures" require of the code?
   A per-round usage record for every review round, solo and fanned (D8); the reading happens over live runs after landing.

## Rejected alternatives

- Literal `cluster-fan` on the recipe rows (the board's proposal): toggling needs a rebuild, so fanned and solo rounds cannot be compared on one deployed binary.
- Recipe literal fan plus a `loom.yaml` on/off switch: two sources for one fact, a new mismatch class between Bouncer and round.
- Per-segment fan model keys (`discussion_fan_review`, `plan_fan_review`): no one asked for different fan models per segment.
- A fan model on the fan entry in `burler.yaml`: run-tuning is kept off the content profile by design, and `loom.yaml` already owns every review model.
- A model per fork: Claude Code forks always inherit the parent's model, and a different model would also lose the shared prompt cache that makes forks cheap.
- Making `lyx config reconcile` add missing fans to `burler.yaml`: reconcile's default merge deletes operator additions, which is why the file is seed-only.
- Refusing at loom start when the configured fan is absent from `burler.yaml`: correct but pushes a hand edit onto every seeded hub, where a per-name template fallback serves the same fan with no operator step.
- Per-round, judge-restated exclusions (today's behavior): a judge that forgets to restate an exclusion silently widens the fan again.
- Letting a union that empties the fan fall through to burler's existing rule: that rule restores the whole fan, the opposite of narrowing.
- Fan on by default: the board orders measurement before the default changes.
- A usage summary CLI verb: a one-off comparison reads the per-round records; a verb adds CLI surface and sandbox coverage for no further need.
- Deleting `TestNoGatedRowAuthorizesForkSubagents`: writer rows must still never fork, which the rewritten test keeps pinned.
- A fan for Webster-Review: forks run no git and the subject is a diff.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Where does the fan choice live? `loom.yaml` per segment (`discussion_fan`, `plan_fan`), threaded by row name to both rows. Rationale: measurement needs a runtime switch, and one value per segment keeps the Bouncer and its round in agreement.
- Q2 (resolved, auto-pick): Default fanned or solo? Solo until measured. Rationale: the board entry says the measurement comes before the fan becomes the default.
- Q3 (resolved, auto-pick): How is the fork model set? Run-wide `fan_review`, default `sonnet[high]`, applied to the fanned reviewer session. Rationale: forks inherit the session's model; operator note of 2026-10-08.
- Q4 (resolved, auto-pick): Lens set per fan? Five lenses each, listed in D3. Rationale: derived from the rubrics' "Also flag" lists and the discussion writer's interview categories, with a `leanness` lens countering the discussion fan's additive bias; matches the `standard` fan's size.
- Q5 (resolved, auto-pick): How do seeded hubs get the fans? Per-name fallback to the embedded template. Rationale: seed-only reconcile never adds them.
- Q6 (resolved, auto-pick): Exclusion semantics? Cumulative union from round 2 within a generation, never emptying the fan. Rationale: monotonic narrowing without relying on the judge's memory.
- Q7 (resolved, auto-pick): What does the judge see? A lens section in the Go-rendered facts file. Rationale: the judge never reads the fan or the artifacts.
- Q8 (resolved, auto-pick): Gate quiescence? Forks allowed on BurlerRound reviewers only; writer rows never fork; shuttle's fork wait is the ground, the fork audit the backstop.
- Q9 (resolved, auto-pick): What does measurement require? A per-round usage record for every review round; no summary verb.
- Q10 (resolved, auto-pick): Track judge cost? No. Rationale: the judge takes under a minute and is the same in both arms.
- Q11 (open, for the operator after landing): When the measurement is read, does the template default flip to `discussion_fan: discussion` and `plan_fan: plan`? Not this task's call per the board entry.
