```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (lens fix-claims).

Claims checked:
- D6: "`Deps.LandLocally` ... still commits the run's status records and marks its pair's push" — **holds.** `recordHeldLanding` commits status at `internal/loomcli/landingdeps.go:959` and `landLocally` marks the push at `internal/loomcli/landingdeps.go:937`.
- D6: "`Deps.RecordLanding`, the dobby's landed-commit record" — **holds** (Notes for the plan writer; `finalize.go:969-970`).
- D6: "rewrites it with the same tip" — **holds.** `runpusher.RecordLanding` replaces the landing by slug (`internal/runpusher/parent.go:721-722`), and the caller passes its own commit.
- D8: the held run's owner checks are "by the reconcile's ended and stopped verdicts alone" — **holds.** See `finalize_owner.go:41-44` and `stuck.go:82-95`. An unreachable origin there logs and goes on.
- D8: "`Lym-Landing` trailer names its slug" — **holds** (`internal/fabriclayout/refnames.go:113-114`).
- D6 criterion: "ahead by owed tips alone" — **holds** for a chain whose first-parent commits are all ledger tips. `AheadByOnly` counts first-parent commits only (`internal/fabricengine/landing.go:204-218`), and `parentAheadByOwed` passes the owed tips (`internal/loomcli/landingdeps.go:1016-1027`).
- D8: "with neither, it is the parent's head, as a held run records it today" — **partly verified.** Fabric's merge has an already-up-to-date outcome (`internal/fabricengine/merge.go:194`, `mergelifecycle.go:210-211`), so an ancestor-held run's head is unchanged. I did not verify a squash-held run, where the parent does not hold the work as an ancestor. That is not a finding.

Findings:

1. LOW. D6, the added sentence "`Deps.LandLocally` on a run whose ledger landing stands ... keeps its records commit while origin lacks it, as `oldestHeldRecords` does today".
   - Summary: the sentence leaves out a second condition. `oldestHeldRecords` keeps the earlier records commit only when the new records commit descends from it, and also only while origin lacks it.
   - Evidence: `internal/loomcli/landingdeps.go:993-995` returns the new records when `!descends`, and `:1005-1011` returns them when origin holds the earlier one.
   - Effect: normally harmless, since a run's records branch only moves forward. Still, the bound as written is looser than the code.

Tool calls made:
- Read `internal/loomcli/landingdeps.go:1005-1029`: the rest of `oldestHeldRecords` and `parentAheadByOwed`, past the span my context held.
- grep `reconcileVerdict`: its body, to confirm the held owner checks judge only the reconcile's verdict.
- grep in `internal/fabricengine/merge*.go` for held-work handling: whether the held path's merge leaves the parent's head unchanged.
- grep for fabric `Merge` and its up-to-date outcome: same question, to find the already-up-to-date path.
