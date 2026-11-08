```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard and existing-code claim in `decision-record.md`.

**Claims attacked**
- D1, a restart after Discussion approval resumes at Raddle-Write through the `undiscussed` gate: held. It is the tapestry Motif-block mechanism and is unchanged.
- D2, Raddle-File's third-failure halt bounds the re-armed perch cycles: held (`contracts/recipes/raddle-recipe.yaml` header, max_bounces above the two-return halt).
- D2 with D4, the perch's approval replaces the operator's: **broken**. D2's own auto-accept key settles a spent budget as an accept.
- D4, operator approval is removed only where the recipe rows hold the split perch: held against a hand-edited recipe. Only embedded recipes name settings (`internal/shedrecipe/doc.go:213`).
- D5, one-task outcome ends done through `EndRaddleEntry`: held (`internal/boardengine/raddlefile.go:265`).
- D6, the run lock admits the conversion "by its shape": **broken**. The shape constrains too little.
- D6, a Raddle-File retry finds the converted entry as filed: held. `isRaddleFiledFrom` needs `Discussed` plus `RaddleProjectBodyPrefix(sketch)`, and sketch equals the arras slug (`raddlefile.go:223-224`).
- D6, a failed write restores the entry's prior record: held as a stated decision. Today's rollback removes every record file it wrote (`raddlefile.go:126-130`), so D6 has to change it, and D6 says so.
- D6, the post-filing status skip "ends when the run's records are cleared": **broken**.
- D6, `lym beam create` clears the done arras run by extending the recipe test: held (`internal/battencli/cleanup.go:158`, `recipe != shedrun.RecipeMotif`).
- D7, the refusal is decided by the rows: **broken** against Scope Out. The raddle recipe holds a `RaddleFile` row too.
- D7, the after-filing way forward "the filed task's own run": **broken**. No such run exists while the part is a draft.
- D8, set-status is the only trigger, and filing and upsert release nothing: held.
- D8, dobby starts released members in `depends_on` order: held (`internal/dobbyengine/starts.go:61-75`, pending dependencies hold a released task).
- D10's rationale in the family outcome, "a dependent would start before the work lands": **broken**.
- D10, a board refusal of the write returns the run to Raddle-Write: **broken** as a bound. No re-split clears that cause.

**Findings**

1. **MEDIUM, design, D6 run-lock admission.**
   The shape admission states no bound on the other fields, so any writer can end the run's hold and rewrite the entry.
   Counterexample: during the Discussion, an agent runs `lym board upsert` on its own slug with kind `project`, no status, `discussed`, and a body of `Groups the entries filed from the Sketch \`<slug>\`` followed by any text. It also changes the title, labels or `depends_on`.
   The admission as worded passes it, and the run lock no longer holds the entry, because it has no run status.
   `endsAsSketch` requires the new entry to equal a derived target in every scope field (`internal/boardengine/sketch.go:74-79`).
   Fix: D6 should admit only a change equal to a derived project-of-before. Title, labels, `depends_on`, `project` and `beam` stay unchanged; the body is exactly prefix, then part list, then former body; `order` is exactly the filed parts.
   D6 should also state that this is what the admission lets through.
   Evidence: `internal/boardengine/runlock.go:162-170`, `scopeChanged` at `runlock.go:208-231`.

2. **MEDIUM, design, D6 post-filing status skip.**
   The skip is keyed on "the run's raddle store", but the raddle store is the board's per-slug directory `raddle/<sketch>/`.
   Nothing outside `boardengine` clears it, and `RaddleStore.Holds` matches on `entry.Sketch == slug`.
   Counterexample: the family outcome files, then the operator runs `lym beam create` on the project. ClearMotifRun clears the run records, but the board store still holds the filing under the slug.
   If the skip reads that store, the beam run's status writes under the slug are suppressed for good. That is the harm the support log gives for rejecting the body-prefix key.
   If the skip reads a run-local record, D6 must say which one.
   Evidence: `internal/boardengine/raddlestore.go:45-51` and `raddlestore.go:84-86`. A grep found no caller of `RaddleStoreDir` that removes the store outside `boardengine`.

3. **MEDIUM, design, D2 auto-accept key with the D4 bound.**
   D2 gives the split segment an auto-accept setting. A spent budget then settles a CONTINUE or CIRCLING round as an automatic accept.
   In `perch` mode that accept routes to Raddle-File with no operator approval, so a proposal with open non-blocking findings files.
   D4's bound does not mention this route, and D2 states no bound for the key: no default, and no mention that it is a guard-category key.
   Evidence: `internal/shedadapters/bouncer.go:145-148` and `bouncer.go:1352`; `internal/reviewbudget/reviewbudget.go:28` and `reviewbudget.go:42`.

4. **MEDIUM, design, D10's rationale in the family outcome.**
   A dependency on a plain project already waits until every landing member is done, and draft member tasks count as landing members.
   So a dependent left on the converted project would not start early, and the rationale holds only for the one-task outcome.
   Moving those dependents still adds board writes and a failure path for no gain.
   Evidence: `internal/boardengine/dependency.go:10-11` and `dependency.go:18-26`; `internal/boardengine/siblings.go:40`.

5. **LOW, design, D10's refusal path.**
   A board refusal of the dependents' re-point returns the run to Raddle-Write "like a moved board", but no re-split changes another entry's run lock.
   Counterexample: a running entry whose `depends_on` names the arras slug. Each retry fails the same way and burns a full perch cycle, up to the third-failure halt.
   The way forward should name the blocking entry rather than re-split.
   Evidence: `runlock.go:166-168`; the return loop in `contracts/recipes/raddle-recipe.yaml`.

6. **LOW, design, D7's after-filing way forward.**
   It names "the filed task's own run", but a filed part is a draft with no run, and `lym loom decision add` on a slug with no run refuses.
   That refusal's way forward is to start the run (`internal/loomcli/decision.go:71`).
   In the family outcome there are several filed tasks, plus the project that D8 says keeps the whole record.
   The way forward should say where the decision goes before release, and to which part.

7. **LOW, consistency, D7's refusal scope.**
   The refusal applies on "a run whose recipe holds a `RaddleFile` row", and the raddle recipe holds one, so plain raddle runs change behaviour too.
   Scope Out says `raddle` keeps its behaviour.
   Either limit the refusal to `perch` mode or name the raddle change in Scope.
   Evidence: `contracts/recipes/raddle-recipe.yaml` producers, `Raddle-File` with engine `RaddleFile`.

**Tool calls**
- Read `scopeChanged` in `runlock.go`, the `clearMotifRun` recipe test in `cleanup.go` and `Store.SetStatus`: my context lacked the scope-comparison body and the recipe check.
- Read `Board.SetRunStatus` and the project-kind checks: I needed to know whether a post-filing status write lands on a no-status project, and it does (`board.go:221-230`).
