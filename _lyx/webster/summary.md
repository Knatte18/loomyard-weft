# Bound batten status history during budget-exempt waits

All four batches finished `done`, and the integration suite reported `OK`.

- **01 history-fold** (`f71445cc3`): `HistoryEntry` gained `repeats` and `last_at`, and `appendOrFold` (with `foldsInto`) landed in `internal/shedengine/run.go`, with `historyfold_test.go`.
- **02 step-fold-wiring** (`9c1889877`): every verdict arm in `run.go` records through the fold, and `activity.last` is recomposed after a fold. A small `recordedVerdict` helper was added, which the card did not name. `docs/overview.md` was updated and `step_fold_test.go` added.
- **03 approval-acted-layout** (`727655ab9`): `approvalActedContent` and `parseApprovalActed` were added in `internal/battenshed/innerrun.go`, with table-driven tests.
- **04 awaiting-resume-pending** (`bc56b0584`): `callAwaiting` records the child's history length in the marker and reports "driver has not re-stepped yet" while it is unchanged. A longer history or an old one-line marker keeps the re-approve hint. Tests were added in `innerrun_awaiting_test.go`.

The only deviations reported were informational scope warnings about new helpers and tests outside the declared glyphs.
