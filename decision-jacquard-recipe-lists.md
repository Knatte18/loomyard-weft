## Goal

Batten, loom verbs and the report admit jacquard where it walks tapestry's path, and cost cells count a discussed tapestry run as jacquard

Takes: jacquard-recipe's shedrun.RecipeJacquard, loomshed.NameRecordSeed and the Jacquard block row. D8 whole but loomrecipe's lists, which jacquard-recipe adds: every non-test closed recipe list naming shedrun.RecipeTapestry for a path a jacquard run walks names jacquard too, found by grepping for it outside tests: batten's admitted child recipes and their refusal text (internal/battencli/wire.go childRecipeSupported and unsupportedChildRecipeRefusal), loomcli's lists (arbiterprompt.go, arm.go, peers.go, wiring.go), tools/tokencount/shed.go, and the report's selected run recipes (internal/reportengine/runs.go); the report's jacquard phase table is tapestry's without its Discussion families, and both tables put the Jacquard block row and Record-Seed in Preflight's phase; a default that falls back to tapestry when a run has no seed (sharedbootstrap.go resolveSeedRecipe) stays tapestry. D9 whole: internal/reportengine's cost cells key a tapestry run whose stored_record is true under jacquard, and only the cell key changes.
Delivers: Batten admits jacquard as a child recipe; lym report selects jacquard runs with their phase table; a tapestry run with stored_record true lands in the jacquard cost cell and the tapestry cell with stored_record true is left empty.
Exports: No new symbol; batten's admitted child recipes include jacquard, which jacquard-start-refusal's batten tests rely on.

## Scope

The part changes these packages:

- internal/battencli (about 20 changed lines)
- internal/loomcli (about 40 changed lines)
- internal/reportengine (about 190 changed lines)
- tools/tokencount (about 5 changed lines)

## Decisions

### D8 — Closed recipe lists admit jacquard where it walks tapestry's path

Every closed recipe list in Go that names tapestry for a path a jacquard run also walks names jacquard too:
batten's admitted child recipes and their refusal text, `internal/loomrecipe`'s recipe lists, and the report's selected run recipes.
The report's phase table for jacquard is tapestry's without its Discussion families.
Both tables put the Jacquard block row and Record-Seed in Preflight's phase.
A default that falls back to tapestry when a run has no seed stays tapestry.

Rationale: a jacquard run builds, lands and reports as a tapestry run of a discussed entry does.

### D9 — Cost cells count a discussed tapestry run as jacquard

`internal/reportengine`'s cost cells key a tapestry run whose `stored_record` is true under jacquard.
Run reports and phase tables keep the recipe the run's seed names.

Bound: only the cost-cell key changes, and only for tapestry runs with `stored_record` true; the tapestry cell with `stored_record` true is left empty.
A tapestry run of an entry flagged discussed whose record file was missing ran Discussion and is still counted under jacquard;
that case is rare, and the cost cell accepts it.
A jacquard run re-seeded as tapestry after Record-Seed's Stuck keeps its first seed's usage record (`internal/loomcli/sharedbootstrap.go`),
so it is counted under jacquard though it ran Discussion; that case is rare too, and the cost cell accepts it.

Rationale: those runs walked jacquard's rows, and raddle's cost lines for jacquard parts would otherwise read too few runs for weeks.

### Added after Discussion (arbiter, 2026-11-28): Jacquard stays one recipe of every row after Discussion; the plan-and-build-recipes note is later work

Decision: Keep D1 as written: jacquard is one recipe holding Preflight, Record-Seed and every tapestry row from Drift-Baseline to Friction-Reflect, and tapestry is the Motif block followed by the Jacquard block (D2). The board note `plan-and-build-recipes` (F3, project `plan-ahead-chains`), which splits those rows into a plan recipe and a build recipe, is not designed or built in this run. It stays a note for later work that may split jacquard. The record's Scope Out list gains no entry for it, since it names no part of this task; no card of this run adds a plan or build recipe, a Y-to-Z pair handover, or a Worktree-Teardown skip.

Rationale: The task's board entry settles it. Its Idea has jacquard hold 'Plan, Plan Perch, Webster, review and landing', and its operator-marked 'Tapestry stays (operator, 2026-11-28)' passage has tapestry made of motif followed by jacquard. The note is a separate, unpromoted note; building its split here would widen the task's scope with new recipes, pair handling and batten teardown changes the entry never asks for. Answers review finding F8 of round 1.

### Added after Discussion (arbiter, 2026-11-28): The jacq nickname is written by the run's parent after landing, as D11 says

Decision: Keep D11 and its Open risks line unchanged. No card of this run writes the 'jacq' nickname or Bolt's `heddle.json`. Once the run has landed, the run's parent adds the nickname line D11 gives to the `nicknames` entry with `lym heddle set`, which completes the board entry's ask that the task add 'jacq' to the repository's nicknames. The Review criterion for D11 stays: no file of the diff writes Bolt's `heddle.json` or a nickname line.

Rationale: `PATTERN-heddle-instructions` has `heddle.json`, which lives at the Bolt checkout's root and not in this worktree, written only through the `lym heddle` verbs, and `lym heddle set` renders into every live worktree of the hub at once. A card writing it before landing would publish a nickname for a recipe the hub's binary cannot run yet, the alternative the support log rejects. The board entry asks only that the nickname be added, not by which step, so D11's route answers it. Answers review finding F9 of round 1.

### Added after Discussion (parent, 2026-11-28): Nickname jacq written after landing

Decision: Accept D11: the parent writes the jacq nickname into the heddle nicknames entry with lym heddle set once the run lands; no card of this run writes it.

Rationale: The heddle glossary is hub-level text the orch keeps; writing it after landing keeps the run's scope to code and avoids a nickname for a recipe that has not landed.

### Added after Discussion (parent, 2026-11-28): Jacquard gains no motif-run clear; a Sketch goes only to raddle

Decision: Apply F16's fix: delete D5's motif-run passage. A finished motif run turns its entry into a Sketch (finishesAsSketch in internal/loomcli/wiring.go), which only a raddle run takes (checkDiscussedPromotion in internal/boardengine/sketch.go), so jacquard builds the parts raddle files and other discussed tasks, and batten gains no motif-run clear for it. D5's Bound drops the clause on reading the slug's done run for the clear; the acceptance criterion on a jacquard start after a done motif run and the Constraints fact on a motif run's task branch are dropped. Batten's refusal keeps its lym batten cleanup --apply clause for a task whose Sketch the board refused. F17 falls with the passage it concerned; nothing else changes. Answers round-3 findings F16 and F17.

Rationale: The board's standing rule, kept by the operator's board policy, is that a Sketch is split only by its raddle task; the entry asks for jacquard on raddle's parts, not for a Sketch to run jacquard directly. The fix narrows D5 to that and removes a recovery sequence that loops on two refusals.

## Constraints

- `PATTERN-shed-recipe-registry`: jacquard declares `contract: loom` in its standard form; the start refusal reads rows, not the name; RecordSeed is one row of the registry map.
- `PATTERN-code-conventions`: the recipe name is one named constant in `shedrun`; the refusal fills it from that constant.
- `PATTERN-refusal-way-forward`: each start refusal site, Record-Seed's Stuck and the decision verbs' RecordWrite-path refusals name their ways forward, and each lands with a test that reaches it.
- `PATTERN-config-layers`: no config key moves; the new profile blocks are template additions the hub reconcile carries, and D7's one-time reinterpretation covers the cases the reconcile leaves.
- `PATTERN-heddle-instructions`: the nickname changes only through `lym heddle set`.
- Fact, from `internal/shedbuild/expand.go`: a block's rows keep their names, segments must not clash with rows outside, and an outermost block row with an empty `on_done` is refused today.
- Fact, from `internal/shedbuild/loomcontract.go`: the loom contract today refuses a recipe with other than one DiscussionWrite or RecordWrite row.
- Fact, from `internal/loomshed/recordwrite.go`: Record-Write writes only a missing record or support log and commits every call.
- Fact, from `internal/boardengine/raddlefile.go` and `internal/loomshed/raddlefile.go`: filing writes each part's record, sets its discussed flag and re-runs the proposal check first.
- Fact, from `internal/runprofile/doc.go`: the reconcile gives a profiles file each template key it holds as its base does, gained ones included.
- Fact, from `internal/loomcli/arm.go` (`resolveRecipe`), `internal/shedengine/status.go` and `internal/loomcli/sharedbootstrap.go`:
  every loom verb reads its recipe from the seed and the status file names none,
  so a re-seeded run goes on under the new recipe from its current row,
  and a `lym loom start` over an existing status file commits the new seed and wakes the driver.

## Auto-mode assumptions

- Config keys stay under `tapestry.*` (D1).
- Preflight runs after Discussion in tapestry (D2).
- The seed is its own engine under a widened loom contract (D4).
- Raddle's `tapestry` shape keeps its name (Scope).
- A jacquard profile block is tapestry's minus `loom.discussion` (D7).
- The nickname is written by the parent after landing (D11).

## Open risks

## Acceptance criteria

- D8: batten admits jacquard as a child recipe, and the report selects a jacquard run with its phase table (tier 0).
- D9: a tapestry run with `stored_record` true lands in the jacquard cost cell, and one with false stays in tapestry's (tier 0, `internal/reportengine`).
