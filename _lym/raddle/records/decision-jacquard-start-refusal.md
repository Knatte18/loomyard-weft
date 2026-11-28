## Goal

Jacquard refuses an entry with no stored record at batten's start and at a first lym loom start, and the decision verbs name the seed row

Takes: jacquard-recipe's RecordSeed engine, Record-Seed row and RecordEngine, and jacquard-recipe-lists' batten admission. D4's decision-verb steps: the decision verbs treat RecordSeed as RecordWrite (rowWritten in internal/loomcli/decision.go), with Record-Seed among the rows before the record is written, and the RecordWrite-path refusals decisionWayRecordStart, decisionWayRecordResume, decisionWayRecordGoto, decisionWayUncommittedGoto and recordWritePending name the row of the run's recipe that runs the record or seed engine, read through loomrecipe.EngineRow, never the literal Record-Write; darn's Record-Write row is unchanged; plus the tier-1 tests of Record-Seed's ways forward. D5 whole: one predicate in internal/loomrecipe (needs a stored record: the expanded rows hold a seed row and no record row), true for jacquard and false for tapestry, darn, motif and raddle; the refusal message `<slug> runs the jacquard recipe, which builds from a stored decision record, and it holds none`, filled from shedrun.RecipeJacquard, judged at batten's start guard beside raddle's check (internal/battencli/startguard.go, way forward the tapestry board upsert, a done slug refused first by doneSlugRefusal) and at a first lym loom start (stat of the status file before loomshed.Seed in sharedbootstrap.go, way forward naming the batten pair's cancel, upsert, set-status and batten run sequence and the other pair's re-seed keeping driver and parent as disagreeingSeedRefusal does); a resume and a re-run of lym loom start over an existing status file pass over it. Per the parent's added entry, batten gains no motif-run clear.
Delivers: Both start sites refuse a jacquard entry with no stored record with their own way forward, before any worktree or status file is made; a jacquard run's lym loom decision add takes the RecordWrite path before and after Record-Seed; the Record-Seed Stuck recoveries for tapestry and for a re-seeded jacquard run work end to end.
Exports: Fixed for later work: the predicate loomrecipe.NeedsStoredRecord(recipe string) (bool, error).

## Scope

The part changes these packages:

- internal/loomrecipe (about 45 changed lines)
- internal/loomcli (about 560 changed lines)
- internal/battencli (about 140 changed lines)

## Decisions

### D4 — A Record-Seed engine seeds the stored record, and the loom contract admits it

A new registry engine, `RecordSeed`, runs as row `Record-Seed`.
It is Record-Write's producer in a stored-only mode: it writes whichever of the decision record and support log is missing, the record from the entry's stored record, and commits the pair.
A record that exists is never overwritten, so an approved record and every entry added after Discussion survive.
With the record missing and no stored record, the row is Stuck with nothing written.
Its reason names a way forward per recipe:

- a jacquard run names two cases, as `lym loom start`'s refusal does (D5):
  - a pair batten made: `lym batten cancel <slug> --remove` from the hub's prime worktree, `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`, `lym board set-status '{"slug":"<slug>","status":null}'`, then `lym batten run <slug>`;
  - any other pair, from its task worktree: `rm <seed path>`, `lym shed seed self --recipe tapestry --driver <driver> --param parent=<parent>`, `lym loom goto --to Discussion-Write`, `lym loom start`, then `lym loom resume`;
    the reason fills in the seed's path and its recorded driver and parent, as `disagreeingSeedRefusal` does,
    and the start commits the new seed before the resume lifts the goto's pause;
- a tapestry run: `lym loom goto --to Discussion-Write`, then `lym loom resume`.

The loom contract in `internal/shedbuild/loomcontract.go` changes from "one record row" to:
a loom recipe holds at most one record row (DiscussionWrite or RecordWrite) and at most one seed row (RecordSeed), and at least one of the two.
When it holds both, the seed row follows the record row on the on_done path from the entry.
Every other clause reads the seed row as the record row when the recipe holds no record row.
`loomrecipe.RecordEngine` keeps returning the record row's engine, so tapestry's decision verbs read Discussion as today, and returns RecordSeed for a recipe with only a seed row.
The decision verbs treat RecordSeed as they treat RecordWrite, with Record-Seed among the rows before the record is written.
The RecordWrite-path refusals in `internal/loomcli/decision.go` (`decisionWayRecordStart`, `decisionWayRecordResume`, `decisionWayRecordGoto`, `decisionWayUncommittedGoto`) and `recordWritePending` name the row of the run's recipe that runs the record or seed engine,
read from the recipe's rows through `loomrecipe.EngineRow`, never the literal Record-Write.
Darn's Record-Write row is unchanged.

Bound: a seed row writes only from the stored record, never from the entry body, and the contract admits it only beside or in place of the one record row.

Rationale: jacquard needs the seed tapestry's Motif gate gives a discussed entry, and the contract needs one row to name the record's source.
Writing only a missing file makes the row a no-op after tapestry's Discussion, so the seed never clobbers the run's record.

### D5 — Jacquard's start refusal is judged by the recipe's rows

One predicate in `internal/loomrecipe` reports whether a recipe needs a stored record: its expanded rows hold a seed row and no record row.
It is true for jacquard and false for tapestry, darn, motif and raddle.
A needing recipe refuses the start of an entry that is not flagged discussed with its record file on the board, the same reading raddle's start guard makes.
The refusal is one message, `<slug> runs the jacquard recipe, which builds from a stored decision record, and it holds none`.
It is judged in two places, and each carries its own way forward:

- batten's start guard, for a run or step with no status file, beside raddle's check, so no worktree is made.
  Its way forward is `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`.
  A slug that holds a done run never reaches the check:
  batten's done-run refusal (`doneSlugRefusal` in `internal/battencli/arm.go`) refuses the start first.
  That refusal names `lym batten cleanup --apply`.
- `lym loom start`, for a start made in a worktree directly, on a first start only:
  the check stats the status file before `loomshed.Seed` and runs only when it is absent.
  The stat judges the check alone; seeding still goes through `loomshed.Seed`'s lock, and a refused start writes no status file.
  That start reads its recipe from the worktree's seed, which a board edit does not change, so its way forward names two cases.
  A pair batten made takes D4's jacquard sequence: from the hub's prime worktree `lym batten cancel <slug> --remove`, `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`, `lym board set-status '{"slug":"<slug>","status":null}'`, then `lym batten run <slug>`.
  Any other pair removes its seed file and re-seeds it with `lym shed seed self --recipe tapestry`, keeping the seed's driver and parent as `disagreeingSeedRefusal` does, then runs `lym loom start` again.

A resume, and a re-run of `lym loom start` over an existing status file, passes over it;
Record-Seed's Stuck guards a run whose entry lost its record before the row ran.

A finished motif run turns its entry into a Sketch (`finishesAsSketch` in `internal/loomcli/wiring.go`);
a Sketch the board refuses leaves the entry a draft task over its done run (`sketchFinish`).
The done-run refusal above guards that task's start.
Only a raddle run takes a Sketch (`checkDiscussedPromotion` in `internal/boardengine/sketch.go`).
So jacquard builds the parts raddle files and other discussed tasks, and batten gains no motif-run clear for it.

Bound: the check reads the board once at start;
it refuses only a needing recipe on an entry with no stored record.

Rationale: `PATTERN-shed-recipe-registry` has a loom verb refuse by a recipe's rows, never its name.

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

- D4: `loomrecipe.RecordEngine` returns DiscussionWrite for tapestry and RecordSeed for jacquard, and `lym loom decision add` on a jacquard run before and after Record-Seed takes the RecordWrite path (tier 0 and tier 1).
- D4: a tapestry run Stuck at Record-Seed moves with `lym loom goto --to Discussion-Write` and runs Discussion on resume (tier 1).
- D4: a jacquard run Stuck at Record-Seed in a pair batten did not make, re-seeded as tapestry, moves with `lym loom goto --to Discussion-Write` and runs Discussion after `lym loom start` and `lym loom resume` (tier 1).
- D4: on a jacquard run past Record-Seed with an uncommitted record, `lym loom decision add` refuses with a way forward naming `lym loom goto --to Record-Seed` (tier 1).
- D5: `lym batten run` of a jacquard entry with no stored record is refused before a worktree is made, and a first `lym loom start` on a jacquard seed of such an entry refuses the same;
  each site's message carries its own way forward, and the test reaches both;
  an entry with a stored record starts, and an undiscussed tapestry entry starts;
  re-running `lym loom start` on a seeded jacquard run whose entry has lost its record is not refused there (tier 1).
- D5: the needs-stored-record predicate is true for jacquard and false for tapestry, darn, motif and raddle (tier 0).
