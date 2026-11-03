```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision — survives or breaks**

- D1 file at Bolt root vs render: `RenderToDisk` deletes only previous-manifest names, so a record file outside the manifest survives — **breaks** when `design_prefix`+slugA spells `decision-`+slugB: the render writes a design doc over the record, and once in the manifest a later render deletes it (F1).
- D1 file vs board sync commit: `commitDirty` → `bolt.Commit` stages the working tree → the record file rides the board commit. Survives.
- D2 flag vs run lock and dates: `checkRunLocks` and `stampDates` decide only through `scopeChanged`, which has no `discussed` (runlock.go:189-213, dates.go:48). Survives.
- D2 done-entry refusal vs replay set-aside: ledger `Runs` sets `Ended: isDone`; a by-entry commit whose runs are all ended is set aside, and the record file belongs to the same replayer (boardengine/replay.go:252, fabricengine/replay.go:689-695). Premise holds.
- D2 write-file-then-flag vs later checks: `checkProjectRules`, `checkNewSlugs`, `checkTaskCreation` run after `fn`; a refusal there leaves file without flag, which D2 already states. Survives.
- D4 vs Rename, RemoveTask, MergeTasks, Prune (the entry-removing Board methods): covered by D4. Survives.
- D4 vs replayed prune commit against origin that ended the entry: whole commit set aside, so the file stays on origin with its entry — consistent with D4. Survives.
- D4 vs `afterSave`: runs under lock after save (board.go:156). Premise holds.
- D5 vs replay `Apply`: files classified only through `boardDesignSlug`; D5 adds the record claim. Apply re-renders after applying; record not in manifest. Survives.
- D5 design-file rule (`applyDesignFile`): writes/removes/reports-lost by base/before/after bytes; fits a plain record file. Premise holds.
- D6 vs Bouncer approving paths: `settle`, `acceptRound`, `approveWithoutReview` all call `approveAndCommit` (Approve before Commit); carry-over written before Approve on CONVERGED/accept; resume path `resettle` re-runs carry-over then Approve. Survives (store idempotent).
- D6 vs `CommitDiscussion`: pathspec is the whole discussion dir, so the working copy at Approve is what the commit lands. Survives.
- D8 vs decision add's commit-failure paths: a commit failure without an amendment refuses with `decisionWayFabricCommit`; a later `lym fabric commit` lands the record but no copy ever runs; on the landed-despite-error path (`afterFailedCommit` landed=true) D8's "after its commit succeeds" is ambiguous — **breaks** (F2).
- D3 vs upsert allowlist: `beam`/`front` stored-value precedent exists (store.go:456-460). Survives.
- Older binary vs `discussed`: unknown key stays as origin holds it in replay (boardengine/replay.go:145); consistent with the Auto-mode assumption. Survives.

**Findings**

F1 — MEDIUM — design — decision-record.md D1
Summary: D1 claims a render never deletes the record file, but its refusal only blocks a `design_prefix` equal to the record prefix; the render writes design files named `<design_prefix><slug>.md` (render.go:795) with no check the name is free. E.g. `design_prefix: ""` with an entry slug `decision-foo`, or `design_prefix: "decision-f"` with slug `oo`, renders the same name as entry `foo`'s record; `RenderToDisk` overwrites it (render.go:49-52), and once that name is in the manifest a later render with that body emptied deletes it (render.go:55-61). The refusal must cover every prefix whose design-file names can coincide with `decision-<slug>.md`, not equality alone.
Evidence: internal/boardengine/render.go:49-61, :795; internal/boardengine/config.go:31 (no validation today).

F2 — LOW — design — decision-record.md D8 (and D9 as the claimed way forward)
Summary: D8 copies only "after its commit succeeds". Decision add's existing failed-commit path refuses with `decisionWayFabricCommit` ("run lym fabric commit") (decision.go:780-782); following that way forward commits the record but never copies it, and the refusal does not name `lym loom decision store`, so the board copy is left stale with nothing pointing at the fix. D8 also leaves unstated whether the landed-despite-error path (`afterFailedCommit` landed=true, decision.go:784-788, :590) copies. D8 should state both paths copy, or that the failed-commit refusal names `lym loom decision store` once the commit lands.
Evidence: internal/loomcli/decision.go:579-605, :780-788.

Tool calls:
1. render.go 19-66 + grep for removals in legacy.go/migrate.go — needed RenderToDisk's write/delete loop, not in context.
2. boardengine/replay.go 268-352 — needed Apply's file classification and re-render, not in context.
3. grep Bolt-dir scanners and entry-removing Board methods — needed every existing remover of an entry, not in context.
4. loomcli/decision.go 575-606 & 760-822 — needed decision add's commit-failure and landed paths, not in context.
