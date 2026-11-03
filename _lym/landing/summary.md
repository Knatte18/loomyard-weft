# Preflight: stop refusing lym's own run records under _lym

Preflight's worktree-clean check refused a run over files lym itself had written under `_lym` while handling a stop.
In the reported case the tackler left `go-settle.jsonl` untracked in the weft, and every re-step stopped on that file until the tackler handed the stop back.
This change decides the pattern once, so no further file of that kind blocks Preflight.

Closes #27

## What changes

- `preflight.CheckResolved` passes the run-records root (`shedrun.RunsRootRel()`) to `fabricengine.Clean` as an exemption.
  The weft-side check ignores uncommitted content anywhere beneath it.
  Everything else under `_lym`, and the whole warp side, still counts.
  `runStateExemptRels` in `internal/preflight/preflight.go` is the one place that names the exempt paths.
- The settle records are now committed with the run's status.
  `battenRunCommitPaths` and `statusCommitPathspec` add the new `shedrun.SettleRecordsRel` when the file exists, so the file no longer waits for a later records commit.
- `fabricengine.Clean` takes variadic anchor-relative `weftExempt` paths and builds an `:(exclude)` pathspec scoped under the anchor.
  `worktreeDirty` and `dirtyReason` take an optional pathspec to carry it.
  With no exemption the probe runs unscoped, as before.

## Design decisions

- The exemption covers the whole run-records root, not a list of file names.
  A new file lym writes there on a stop or settle path is exempt without a further change.
- Settle records are committed as well as exempt, because they are durable state a resumed machine may need.
  The other run-record files, the stop and condition-stop markers among them, stay uncommitted until a later records commit or the run's removal.
  They are exempt only so lym does not block on them.
- The exemption applies to the weft side only.
  A hub anchored at a subpath is handled by scoping the exclude pathspec under the anchor.

## Tests

- `TestCheckResolved_RunStateUnderLymIsExempt` (preflight integration, tier 1) writes the run-directory files lym's stop and settle paths produce, untracked, for a root anchor and a `backend` anchor.
  It asserts that unscoped `fabricengine.Clean` sees them dirty, that `CheckResolved` passes, and that a file under `_lym/plan` outside the root still fails worktree-clean.
- Unit tests cover the settle-records path in `battenRunCommitPaths`, `statusCommitPathspec` and `SettleRecordsRel`.

## To check by hand

- The tier-1 integration test was not run as part of writing this description; run `lym gate test ./internal/preflight` before landing.
- Open finding `exempt-files-not-carried-by-named-commit` (consistency, medium): run-record files other than the settle records are exempt from the check but not carried by any named status commit.
  Check that leaving them uncommitted until the records commit or run removal is acceptable.
