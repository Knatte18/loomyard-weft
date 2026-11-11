```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The fixes cover every member of each set I searched. Each search was one call per finding, and the three findings were not all set-premised the same way.

- **F1 (every `lockRecord()` refusal needs a test):** `lockRecord()` is called at `decision.go:883` (add), `1149` (store) and `1258` (replace). `decisionWayLock` is used at `883` and `1260`.
  - Add is now reached by the new row in `decision_test.go:1076`.
  - Replace is reached by the `LockFails` row at `decisionreplace_test.go:265`.
  - Store is covered by its existing `LockFails_RefusesWithTheRetryWay` row, as the review states. I did not re-open that test.
- **F2 (every `writeRecord` failure branch needs a test):** `writeRecord` is called at `decision.go:1334`, `1335` and `1397`. The new fake fields are used only by the two new rows, at `decisionreplace_test.go:270` and `275`. The `1397` site is the restore after a commit failure, which `CommitFailsWithoutLandingAndTheRestoreFails` already reaches. So `1334` and `1335` were the only unreached sites.
- **F3 (comments that call add the only way):** `runlock.go:67` and `doc.go:472` now name both verbs. The other sites that mention both verbs or are unrelated, none stale:
  - `pawl-recipe.yaml:15`
  - `overview.md:814`
  - `loom-directive-project-record.md:20`
  - `PATTERN-run-records`
  - `runlock.go:130` is the refusal message, which the review says may keep naming add.
  - `loom-discussion-spec.md:124` and `126` say only add appends the `Added after Discussion` heading and bullet suffix. That stays true, because replace rewrites an entry instead of appending one.

Calls made:
1. Grepped the non-test code, docs and recipes for "decision add" next to "alone", "the one way" or "only", to rerun F3's search across the repo.
2. Grepped `decision.go` and the test files for `lockRecord()`, `decisionWayLock`, `lockErr` and `lockOverride`, to rerun F1's search. My context lacked the call-site list.
3. Grepped `decision.go` and the test files for `writeRecord(`, `writeFirstErr` and `writeAllErr`, to rerun F2's search. My context lacked the call-site list.
4. Grepped `boardengine`, `boardcli` and the stencils for "decision add", to catch phrasings the first F3 search missed.
