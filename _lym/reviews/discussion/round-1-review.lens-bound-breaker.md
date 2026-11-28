```yaml
lens: bound-breaker
outcome: reported
```

**Bound-breaker list** (each claim/bound attacked → held/broken)

1. D2 "an expansion reads none of a block's top-level keys" → held (`expand.go` doc + `Expand`).
2. D2 run parked inside Motif goes on to Preflight and Record-Seed, "which finds the approved record and writes nothing" → held (`recordwrite.go` `Call` writes only missing files).
3. D2 run parked at Preflight: Record-Seed seeds "which is what the Motif gate would have done" → held when no record exists (gate's `seedDiscussedRecord` overwrites unconditionally, seed writes missing only; same result at Preflight).
4. D2 history entries carry no block; checklist joins by name, only label differs → held (no reader outside the engine branches on history `Block`; grep found none).
5. D2 rationale "Preflight's checks guard config the build rows read, which Discussion does not" → broken.
6. D3 bound "the edge lets an empty `on_done` end the run only inside a block that is the parent's last row" → held for routing; a gated last block breaks block-gate skip semantics.
7. D3 "for a loom recipe the terminal check still holds" → held (`checkFinalizeEndsInTerminals`).
8. D4 contract order clause "seed row follows the record row on the on_done path" → held for tapestry (Motif→Discussion-Write→…→Jacquard→Preflight→Record-Seed).
9. D4 "decision verbs treat RecordSeed as they treat RecordWrite" → broken (ways forward name a row jacquard lacks).
10. D4 tapestry Stuck way forward `goto --to Discussion-Write` → held (`CheckGotoTarget` admits backward goto; Discussion-Review segment never settled on that run).
11. D4 jacquard Stuck way forward (cancel --remove, upsert, release, run) → held.
12. D5 "the same reading raddle's start guard makes" (discussed flag + record file) → held, matches `ReadStored`.
13. D5 way forward from `lym loom start`'s refusal → broken.
14. D5 "A resume passes over it" (for the `lym loom start` site) → broken/unestablished.
15. D5 "an entry with a stored record starts" (motif-then-jacquard same slug) → broken.
16. D6 bound "nothing is filed under tapestry" → held (filing re-runs check).
17. D7 "existing hub's profiles file gains the blocks at its next reconcile" → held for template profiles (`FollowTemplate` descends per key; absent-in-base-and-file key is gained); not for operator-added profiles.
18. D9 bound "only tapestry runs with `stored_record` true … those runs walked jacquard's rows" → broken at edge.
19. Constraint facts (expand, loomcontract, recordwrite) → held.

**Findings**

- **BLOCKING · design · `_lym/discussion/decision-record.md` D5** — The way forward named for `lym loom start`'s refusal (`lym board upsert … "recipe":"tapestry"`) cannot lead to a start.
  The run's recipe comes from the seed already in the worktree: `resolveSeedRecipe` returns `existing.Recipe` (`internal/loomcli/sharedbootstrap.go:275-280`, read at `:105-108`), written by batten's Seed-Child (`internal/battencli/wire.go:1672`).
  A board edit leaves the seed at jacquard, so re-running `lym loom start` refuses again; and a rewrite would hit `ErrDisagreeingSeed` (`sharedbootstrap.go:116-120`).
  The record says that way forward is one "a start with no status file can take" — true only at batten's site.
  Fix: name a way forward per site.

- **BLOCKING · design · D4 ("The decision verbs treat RecordSeed as they treat RecordWrite")** — The `checkDarnRecord`/`missingRecordRefusal` refusals a jacquard run would reach carry ways forward naming `lym loom goto --to Record-Write` (`internal/loomcli/decision.go:88`, `:90`) and "once Record-Write has run" (`:86-87`), and `recordWritePending` keys on `loomshed.NameRecordWrite` (`decision.go:604-606`).
  Jacquard holds no Record-Write row, so the goto is refused "names no producer in the list" (`internal/shedengine/goto.go:169-175`).
  With only `rowWritten` named as changing, these refusals ship with a dead way forward, against `PATTERN-refusal-way-forward`; D4 does not say they change.

- **MEDIUM · design · D5 / Acceptance criteria ("an entry with a stored record starts")** — The case the board entry names (motif run then jacquard on one entry) is refused at start: the slug's done motif run makes `refuseAtStart`/`checkRunStatus` return the "has already completed" refusal (`internal/battencli/dobbyreads.go:488-497`, `internal/battencli/arm.go:945-949`), cleared only for a raddle entry (`startClearsMotifRun`, `internal/battencli/startguard.go:178-182`).
  The record does not state whether jacquard gets the same clear, or the operator runs `lym batten cleanup --apply` first.

- **MEDIUM · design · D3** — A last outermost block row with an empty `on_done` and a gate other than `always` enters its block when the gate does not hold.
  Expand emits the block row with `SkipTo: blockExit` (empty here) and `OnDone: entry` (`internal/shedbuild/expand.go:96-104`); `skipTarget` falls back to `OnDone` (`internal/shedengine/producer.go:130-134`), so a skip routes into the block instead of past it or to the run's end (`internal/shedengine/entry.go:56-75`).
  D3's bound does not cover this; fix refuses that shape or has an empty exit end the run.

- **MEDIUM · design · D5 ("A resume passes over it", `lym loom start` site)** — The check sits "before it writes the seed", but bootstrap writes the shedrun seed and runs `loomshed.Seed` on every start including re-entry (`sharedbootstrap.go:105-124`); only `seedErr` tells a first seed from a re-entry (`sharedbootstrap.go:124-127`, `:149`).
  The record does not say the check sits on the first-seed path, so placed at "before the seed" it refuses a re-entry of a jacquard run whose entry has since lost its record, against "A resume passes over it".

- **MEDIUM · design · D2 Rationale ("Preflight's checks guard config the build rows read, which Discussion does not")** — Besides batcher and drift, Preflight runs the stale-stamp hub reconcile and the worktree-clean, fabric-sync, fabric-ready and junction checks (`internal/preflightshed/preflight.go:36-63`, `:226-246`), and Discussion-Write commits `_lym` through that fabric (`commitDiscussion`).
  An undiscussed tapestry run now meets a broken junction, dirty tree or unreconciled config only after Discussion. Motif already runs this way, but the premise as stated is false; fix restates the rationale against what Preflight checks or settles what Discussion loses.

- **LOW · design · D7** — An operator-added profile holding a tapestry block gains no jacquard block (template does not hold it; `yamlengine.FollowTemplate`, `internal/yamlengine/follow.go:41-52`), so a jacquard part naming it is refused by raddle's check, which needs a block for the recipe (D7 Rationale).
  The record covers only "each profile of template.yaml". The same follow also gains `defaults.jacquard: solid` on a hub that set `defaults.tapestry` to another profile, so the two defaults diverge silently.

- **LOW · design · D9 Bound/Rationale** — `stored_record` is the entry's discussed flag at first seed (`sharedbootstrap.go:245`, `:149`), not proof Discussion was skipped.
  A tapestry run of an entry flagged discussed whose record file is missing runs Discussion (gate holds because `seedDiscussedRecord` finds nothing, `internal/loomcli/discussionrecord.go:64-67`) yet is keyed under jacquard, so "those runs walked jacquard's rows" does not hold for it.

**Tool calls**
1. `CheckGotoTarget` and loom goto guards — to check the tapestry Stuck way forward's backward goto is admitted; context lacked goto's checks.
2. `sharedbootstrap.go` seed write path and `writeRunUsage` callers — where `lym loom start` writes the seed and when the usage record is written; context had only `resolveSeedRecipe` lines.
3. grep battencli for done-run refusal — what a start over a done motif run does.
4. `DobbyTasks` in `dobbyreads.go` — the start refusal for a released task over a done run.
5. `doneSlugRefusal` — its text/way forward.
6. grep for readers branching on history entry `Block` — to test D2's history-join claim.
