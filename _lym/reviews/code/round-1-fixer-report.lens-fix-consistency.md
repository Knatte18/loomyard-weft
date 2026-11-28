```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F4 change set.

**LOW**: `internal/landingshed/deps.go:260`, the untouched twin of the F3 narrowing.
- **Summary:** the `Deps.RecordLanding` contract still says it "records commit as the run's landing commit".
- **Evidence:** the changed `recordLanding` doc (`finalize.go:959-960`) now says it hands `Deps.RecordLanding` "the parent's head right after the parent-side merge", which is the run's own landing commit only on a pass whose merge commits. The `Deps.LandLocally` doc (`deps.go:174-176`) says the same of the same value. On a re-run over a parent that already holds the work, `RecordLanding` receives the parent's head, so its field doc overstates it the way F3 described for `LandLocally`.

I found no other disagreement in `landingdeps.go`, the two tests or `_lym/landing/summary.md`.

Tool calls:
1. One `grep` for landing-commit, tip and `writeLedgerLanding`/`LandLocally` mentions across `internal`, `docs` and the root markdown, plus `cat _lym/landing/summary.md`. My context lacked the repo-wide twins and the edited summary as a whole.
