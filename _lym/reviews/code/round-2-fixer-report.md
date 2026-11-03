# Fixer-report, code review round 2

## Changes

- F1 (MEDIUM), fixed as proposed, commit `147cda81`.
  `runStateExemptRels` in `internal/preflight/preflight.go` and the paragraph in `internal/preflight/doc.go` no longer name a pair-teardown commit.
  They state that the status commits stage the settle records, that the other run-record files, the stop and condition-stop markers among them, stay uncommitted until a later records commit or the run's removal, and that the check ignores them as in-flight bookkeeping.
  The "decided in one place" sentence stays.
- F2 (LOW), fixed as proposed, in `_lym/landing/summary.md`.
  The summary now says both status commits (`statusCommitPathspec`, `battenRunCommitPaths`) stage the settle records through `shedrun.SettleRecordsRel`, and no longer presents committing the settle log as an option not taken.
  "The landing's records commit" is gone; the summary states the real carrier.
  Not committed: the fix instructions forbid any git command against a `_lym` path, so the edit sits in the working tree for the loop that owns `_lym`.
- F3 (LOW), fixed as proposed, commit `f0a4774c`.
  The `RunsRootRel` comment in `internal/shedrun/paths.go` no longer names fabricengine's Add or Preflight.
  The `SettleRecords` comment says a status commit stages it through `SettleRecordsRel`.
- Self-check follow-up, uncommitted at report time and committed right after: the `RunsRootRel` sentence in `internal/shedrun/doc.go:45` named fabricengine's Add as its reason, the twin of the F3 comment, and now matches it.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-consistency, LOW, summary lists settle records among exempt files while status commits now stage them: disputed, premise false.
  The settle log is appended after the halt's status commit (`summary.md` line 7), so it is still uncommitted when Preflight runs, and the exemption covers that window.
- fix-consistency, LOW, "a later records commit" is vague: fixed as proposed by the review's own `fix` key, which names exactly this wording; no fix carrier exists to name, so the wording stays hedged.
- fix-affected-set, LOW, `internal/shedrun/doc.go:45` enumerates a consumer: fixed, see Changes.

## Disputed

- fix-consistency LOW (settle records listed as exempt): premise false, evidence above.

## Deferred

Nothing deferred.
