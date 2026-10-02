---
round: 1
focus:
  - "Card 1 (entry-schema) against Decision 7: settle whether the tier rule as card 1 rewrites `validateWrite` and `SetStatus` reaches every write path the decision names (upsert tier changes, upsert-batch, merge, set-deps, card 2's promote, a set-status reopen), in both directions, with the `done` exemption applied the same way on each side."
  - "Cards 3, 4 and 5 against Decisions 8–10 and their acceptance criteria: settle whether a slug in both legacy files refuses the load on read paths too (the criterion says load; card 4 tests only a write), whether every read path (`GetTask`, `ListTasksBrief`, card 5's `Find`) folds in memory and persists nothing, and whether `RetireLegacy`'s fold, save and delete order matches Decision 10."
  - "Whole plan against `_lyx/discussion/decision-record.md`: settle whether every Decision, every Constraint (Bolt never naming the tracker, semantic line breaks in renderer-generated markdown, `Short` on hidden aliases, hub-level tests through `internal/hubforge`) and every acceptance criterion is carried by some card, and whether any card adds scope the record does not license."
  - "Granularity of cards 1 and 4, and the split between cards 2 and 5: settle whether each is one independently reviewable and testable unit, given that card 1 bundles schema, validation, the tier dependency rule and the battencli recipe rename, and card 4 bundles the store-file switch, migration wiring, the two-store facade collapse and the notes call-site repoint, while `store.go` and `render.go` are edited by several cards."
  - "`ImpactSummary` on every Edit card (1, 2, 4, 5, 6, 7, 8): settle whether each states a blast-radius conclusion grounded in the code's callers rather than restating `Intent`, including card 6's `list` order change and card 4's deleted `*Note*` methods and changed `Render`/`RenderToDisk` signatures."
---

Spot checks run before seeding, none promoted to an entry:
the board sync stages through `gitrepo.StageAllAndCommit`, so the legacy-file deletions reach weft without a sync edit, and card 5's sync integration test covers it;
`ListTasksBrief`, `Render`, `RenderToDisk` and the `*Note*` methods have no caller outside `internal/boardengine` and `internal/boardcli`;
`internal/ideengine`'s tests seed a legacy `tasks.json`, which card 4's migration-on-load reads, and the untagged `go test ./...` runs them;
the `ly` plugin discovers skills from its `skills/` directory, so card 9 needs no manifest edit;
the verify section runs integration-tagged tests for every targeted package, `internal/boardengine/boardtest` included.
