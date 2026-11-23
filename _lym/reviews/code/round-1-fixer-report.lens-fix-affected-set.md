```yaml
lens: fix-affected-set
outcome: reported
```

No findings, apart from one stale sibling of F14.

**Findings**

- LOW, `docs/overview.md:848` (F14's set, the module line at `:294` is fixed). The beam bullet still says dobby "spawns the board-tidy agent every few landings" and omits the describer agent. The review named only line 294, and the same fixer-added sentence also describes the beam section at line 837. Evidence: `grep "spawns the board-tidy"` returns both `docs/overview.md:294` (updated) and `docs/overview.md:848` (not updated).

**Set checks that came back clean**

- **F4 sites that stop or keep the tidy agent:** every one now handles the describer too: `recordStart` (`loop.go:477`), `readKeptAgents` (`:515`), `rememberKeptAgents` (`:533`), `exitWorktreeGone` (`:546-559`), the call site (`:604-609`) and the `PrimeTidyShuttle` doc comment. No other `keptTidy` or `PrimeTidyShuttle` reader exists.
- **F5 return paths after the record:** `recordDescription` has two, the landing record read (`describe.go:83`) and the pull-request update (`:91`). Both wrap in `afterRecordError`, and `dobbydeps.go:351` is the only consumer of the changed `Record` signature.
- **F2 callers of `describeStep`:** the only non-test caller is `tick` (`loop.go:730`), which now logs the error through an episode instead of returning it. `spawnDescribe` is called only from `describeStep`.
- **State record fields:** `OwedLine` has no other consumer or doc to update. The only prose listing state fields is `docs/research/loom/lym-state-map-2026-11-15.md:144`, a dated research snapshot that names neither describe field.

**Tool calls**
1. A grep of non-test Go and Markdown files for the tidy-only wording and the `PrimeTidyShuttle`, `keptTidy`, `readKeptTidy` and `rememberKeptTidy` names (F14 and F4 sets). My context lacked the sibling sites of `PrimeTidyShuttle` and the tidy-only overview wording.
2. A grep of `internal` for `afterRecordError`, `BeamDescribe.Record` and `.Record(ctx` (F5 set). My context lacked all consumers of the changed `Record` signature.
3. A grep of `internal` for `describeStep(`, `spawnDescribe(` and `settleDescribe(` (F2 set), to find every caller of the changed error behaviour.
4. A grep for `landed_since_describe` and `landed_since_tidy`, and a second for the worktree-gone wording (F4 and `OwedLine` docs). My context lacked any doc that lists the state record's fields.
