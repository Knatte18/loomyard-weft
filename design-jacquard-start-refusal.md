# Jacquard refuses an entry with no stored record at batten's start and at a first lym loom start, and the decision verbs name the seed row

`jacquard-start-refusal` · Task · feature, loom, batten · draft

Decision record: [Sketch](decision-jacquard-start-refusal.md)

Depends on: [`jacquard-recipe-lists`](design-jacquard-recipe-lists.md)

Filed from the Sketch `recipe-build-from-sketch`.

Takes: jacquard-recipe's RecordSeed engine, Record-Seed row and RecordEngine, and jacquard-recipe-lists' batten admission. D4's decision-verb steps: the decision verbs treat RecordSeed as RecordWrite (rowWritten in internal/loomcli/decision.go), with Record-Seed among the rows before the record is written, and the RecordWrite-path refusals decisionWayRecordStart, decisionWayRecordResume, decisionWayRecordGoto, decisionWayUncommittedGoto and recordWritePending name the row of the run's recipe that runs the record or seed engine, read through loomrecipe.EngineRow, never the literal Record-Write; darn's Record-Write row is unchanged; plus the tier-1 tests of Record-Seed's ways forward. D5 whole: one predicate in internal/loomrecipe (needs a stored record: the expanded rows hold a seed row and no record row), true for jacquard and false for tapestry, darn, motif and raddle; the refusal message `<slug> runs the jacquard recipe, which builds from a stored decision record, and it holds none`, filled from shedrun.RecipeJacquard, judged at batten's start guard beside raddle's check (internal/battencli/startguard.go, way forward the tapestry board upsert, a done slug refused first by doneSlugRefusal) and at a first lym loom start (stat of the status file before loomshed.Seed in sharedbootstrap.go, way forward naming the batten pair's cancel, upsert, set-status and batten run sequence and the other pair's re-seed keeping driver and parent as disagreeingSeedRefusal does); a resume and a re-run of lym loom start over an existing status file pass over it. Per the parent's added entry, batten gains no motif-run clear.
Delivers: Both start sites refuse a jacquard entry with no stored record with their own way forward, before any worktree or status file is made; a jacquard run's lym loom decision add takes the RecordWrite path before and after Record-Seed; the Record-Seed Stuck recoveries for tapestry and for a re-seeded jacquard run work end to end.
Exports: Fixed for later work: the predicate loomrecipe.NeedsStoredRecord(recipe string) (bool, error).

The part changes these packages:

- internal/loomrecipe (about 45 changed lines)
- internal/loomcli (about 560 changed lines)
- internal/battencli (about 140 changed lines)
