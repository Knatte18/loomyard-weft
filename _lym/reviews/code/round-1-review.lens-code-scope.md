```yaml
lens: code-scope
outcome: reported
```

Scope trace of the diff against the decision record's Goal (lens: code-scope).

**Decisions (Goal points; the record has no `Added after Discussion` entries):**
- G1, Idea: sort each hit into a refusal (keep) or a migration or compatibility path whose data no hub still holds (delete).
- G2, Decided: all migration code goes.
- G3, Decided: `lym board retire-legacy` goes, together with its `tasks.json`/`notes.json` migration.

**Per change, the decision it traces to:**
- C1. Board: the `retire-legacy` verb, the `tasks.json`/`notes.json` migration, the `legacy_done` fold, the tier/type conversion, the `draft_status`/`discussed` migrations and the `.gitignore` lock-pattern strip (013, 014–016) → G3, G2.
- C2. Config: every module `Migrate`/`MigrateHubBase`, `KeyMoves`/`KeyCopies`, the warp/weft fold-in, former template values, the base-sparse strip, the worktree sparse migration, the hub-wide copy retirement, `Spec.MigrateLayer`, the concrete reinterpretations and the yamlengine helpers (008, 018–026, 030, 033, 043, 048–050, 093 and others) → G2.
- C3. The silent drop of a retired key on load is gone; such a key is now an unknown key → G2 (it was a compatibility tolerance).
- C4. Refusal texts reworded so they no longer promise a reconcile migration (`internal/batcher/config.go` `retiredKeyError`, shuttle, loom) → forced by C2.
- C5. The runprofile template sets plan keys under `tapestry:`, and the new `recipeRenames` former-name refusal replaces the reconcile's block rename (074) → forced by C2. The refusal itself is licensed by G1's "refusal stays".
- C6. Fabric: the lymcwd Bolt-marker fallback, the `_portals`/`_launchers` sweeps, the cleanup segment, the warp-binding backfill, the `self` run-dir fallback, the legacy exclude lines and the root `.scratch` link (017, 035–040, 060, 080) → G2.
- C7. Shed, loom and pawl: retired rows, `RetiredMove`, the Merge-In fallbacks, the scratch-record moves, legacy verdict words, retired failure kinds and `--no-attach` (053–059, 066, 072, 075–080, 090) → G2. Refusing a recipe's `retired:` map in `internal/shedbuild/parse.go` is forced by this change.
- C8. Dobby and pause: `fixtries`, kept-cancel marking, the adopt/adopt-hold/owner pause callers and `RecordPauseIntent` (031–032, 042, 064–065, 087–088) → G2. The `boardengine.WriteHold` test-only allowlist entry (`cmd/lym/testonlycode_linux_test.go:52`) is forced by this change.
- C9. Shuttle, reed, creel, Slack, orch, heddle (029, 044, 061–063, 071, 081–086) → G2.
- C10. Docs, `pattern/*.md`, the sandbox F17 scenario and `doc.go` files (003, 004, 094, 095) → forced by C1–C9.
- C11. Commit 467b8093 makes `fabric add` commit a name whose only change is a staged deletion → forced by C2: removing the sparse marker exposed the gap.
- C12. The test-only reinterpretation seam (`internal/hubreconcile/export_test.go:20`, test file) → forced by C2. It keeps the mechanism `PATTERN-config-layers` requires under test.
- C13. Commits c3c81cef, 4f8b8fce and 525223fc break over-long comment lines → forced by the comment lint on the comments C1–C9 touch.

**Per decision, the changes that build it:**
- G1: C1–C9, with C4 and C5 keeping refusals.
- G2: C1–C9, built only partly (see F1 and F2).
- G3: C1, fully built: the verb, its help row, its CLI step, `doc.go` and the sandbox no longer name it.

**No change traces to no decision**, so there is no unrequested-scope finding.

**Findings**

- **F1.** MEDIUM · scope · `internal/fabricengine/resetsubpathcheck.go:73`
  - Summary: G2 is not built on the clone path. `recordedAnchorOnWeft` still adopts the retired hub-wide Bolt `.lym-anchor` from `weft:main` when the prime's weft branch has none, and `clone.go:140` still describes it ("else the Bolt legacy marker on weft:main").
  - Why it matters: `pattern/PATTERN-cwd-resolution.md` now states "With none recorded, the anchor is `"."`". The change description claims the "hub-wide Bolt anchor marker retirement and read fallback" is gone, and does not list this read among its kept items.
  - Evidence: `internal/fabricengine/resetsubpathcheck.go:73-78` and `:98-106` (the `branch --show-current` == warpBranch fallback returns `"on weft:main"`); `internal/fabricengine/clone.go:139-141`.
- **F2.** LOW · scope · `internal/fabricengine/mergelifecycle.go:17`
  - Summary: G2 leaves compatibility paths for merge records an older binary wrote, though no hub holds such a record.
  - The paths: the weft arm of `concludeMergeSides` for an older binary's `"staged"` WeftOutcome, and the exemption for a record that predates the recorded source SHAs.
  - The change neither deletes them nor names them as kept.
  - Evidence: `internal/fabricengine/mergelifecycle.go:17`; `internal/fabricengine/doc.go:2013`, `:2094`, `:2133`.
- **F3.** LOW · scope · `internal/hubreconcile` (the per-worktree commit leg)
  - Summary: G2's deletion leaves hubreconcile's per-worktree commit leg with no production input; the summary says only the sparse migration fed it. The diff defers it ("Left for a follow-up") rather than removing it, so the migration's machinery outlives the migration.
  - Evidence: `_lym/landing/summary.md`, "Left for a follow-up" bullet.

**Tool calls:**
1. grep for remaining `migrat`/`legacy`/`older binary` sites in non-test Go, plus the location of `SetReinterpretationModulesForTest`: needed to judge whether G2 is built everywhere, which the diff pieces in context do not show.
2. Read `mergelifecycle.go:10-30`, `resetsubpathcheck.go:60-95`, `fabricengine/doc.go` spans, `beamcli/doc.go:50-56` and `clone.go:134-144`: to classify the remaining hits the grep found.
3. Read the rest of `recordedAnchorOnWeft`, grep beamcli/battencli for `legacy`, grep `mergelifecycle.go` for `staged`: to confirm the weft:main marker read is live code, not only a comment.
4. grep beamcli/battencli for the "legacy beam with a marked pair" path: to check whether a beam compatibility path exists. No code matched, so nothing is filed.
