## Goal

Raddle names jacquard for the parts it files, and the glossary and docs name jacquard and tapestry's two blocks

Takes: jacquard-run-profiles' jacquard blocks, and jacquard-start-refusal's tree with jacquard admitted by batten. D6 whole: internal/raddleparser's part recipe vocabulary becomes jacquard or darn, a part naming tapestry is a recipe-invalid finding whose detail names jacquard as the recipe for a part filed from a Sketch; the raddle stencil (contracts/stencils/raddle/raddle-template-agent.md), contracts/specs/raddle-proposal-spec.md, the raddle review rubric (contracts/stencils/loom/loom-rubric-raddle-review.md) and the loom raddle context's profile listing (internal/loomcli/raddlecontext.go) say jacquard where they name the part recipe tapestry; the tapestry shape keeps its name; filing copies the part's recipe, tested with a filed jacquard part. D10's glossary and docs steps: the jacquard line in internal/heddleengine/glossary.md as D10 gives it, a jacquard row in docs/reference/weaving-glossary.md with its tapestry row saying tapestry is motif followed by jacquard, and docs/overview.md's recipe mentions; the recipe-header step landed in jacquard-recipe. D11: no file of the diff writes Bolt's heddle.json or a nickname line.
Delivers: Raddle proposals name jacquard or darn for each part and are refused with a jacquard-naming detail when a part names tapestry; filed parts carry jacquard, their record and the discussed flag; the glossary and docs list jacquard.
Exports: raddleparser.RecipeJacquard beside RecipeDarn; RecipeTapestry leaves the part vocabulary.

## Scope

The part changes these packages:

- internal/raddleparser (about 30 changed lines)
- internal/loomcli (about 10 changed lines)
- internal/loomshed (about 20 changed lines)
- contracts/stencils (about 5 changed lines)
- contracts/specs (about 10 changed lines)
- internal/heddleengine (about 2 changed lines)
- docs (about 40 changed lines)

## Decisions

### D6 — Raddle names jacquard for the parts it files

`internal/raddleparser`'s part recipe vocabulary becomes jacquard or darn.
A part naming tapestry is a `recipe-invalid` finding whose detail names jacquard as the recipe for a part filed from a Sketch.
The raddle stencil, `contracts/specs/raddle-proposal-spec.md`, the raddle review rubric and the loom raddle context's profile listing say jacquard where they name the part recipe tapestry.
Filing copies the part's recipe, so a filed part carries jacquard with its stored record and the discussed flag, which filing sets today.

Bound: filing re-runs the check, so a proposal approved before this change with a tapestry part is sent back at filing, rewritten to jacquard and approved again; nothing is filed under tapestry.

Rationale: a part filed from a Sketch always has its record, so naming tapestry leaves whether Discussion runs to a flag.

### D10 — The glossary and docs name jacquard

`internal/heddleengine/glossary.md` gains the line:
`- **jacquard**: the recipe that builds a task from its stored decision record: tapestry's rows after Discussion.`
`docs/reference/weaving-glossary.md` gains a jacquard row, and its tapestry row says tapestry is motif followed by jacquard.
`docs/overview.md`'s recipe mentions and the tapestry and motif recipe headers name jacquard where they list recipes or describe tapestry's rows.

Rationale: the operator asked for the glossary term, and the docs list each recipe.

### D11 — The "jacq" nickname is written through heddle after landing

The nickname line `- Read "jacq" as the jacquard recipe: tapestry's rows after Discussion, run on an entry with a stored decision record.` joins the `nicknames` entry of Bolt's `heddle.json`.
The run's parent writes it with `lym heddle set` once the run has landed; no card of this run writes it.

Rationale: `PATTERN-heddle-instructions` has `heddle.json` written only through `lym heddle`, and it is the repository layer in Bolt, not a file of this worktree.

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

- D11 asks the run's parent to run `lym heddle set` for the `nicknames` entry once the run lands; the entry asks the task to add it, and no card of the run can.

## Acceptance criteria

- D6: raddle's check refuses a part naming tapestry with a detail naming jacquard and accepts jacquard and darn; a filed jacquard part carries recipe jacquard, the discussed flag and its record (tier 0).
- Review: D6 — the raddle stencil, `contracts/specs/raddle-proposal-spec.md` and the raddle review rubric name jacquard as the part recipe.
- Review: D10 — `internal/heddleengine/glossary.md` carries the jacquard line, and `docs/reference/weaving-glossary.md` and `docs/overview.md` name jacquard and tapestry's two blocks.
- Review: D11 — no file of the diff writes Bolt's `heddle.json` or a nickname line.
