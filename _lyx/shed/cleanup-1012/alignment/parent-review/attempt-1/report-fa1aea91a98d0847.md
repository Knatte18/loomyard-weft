---
verdict: APPROVED
---

## Scope against the board entry

The design covers each of the entry's three sections, and adds nothing from another entry.

- tokencount cutoff skew: D1 adds the opt-in `-no-cutoff` flag the entry's idea names.
  The entry said to add it if the `tokencount-full-runs` measurements showed the skew; no such entry is on the board, and the record lists adding the flag now as an auto-mode assumption.
  Lifting the shed span's `Older` clamp follows from the same tag date and is listed as an assumption too.
  The tolerance window, the entry's alternative, is rejected with a reason.
- Audit reader globs: the entry says readers still take unquoted globs and free flags.
  D2 rejects an unquoted parenthesis, the one reader-glob path that can run code, and documents in `internal/fabricengine/doc.go` why other globs and flags need no bound.
  It also documents the git `--` pathspec reason the entry asks for.
  Goal 2 states that the brief's "bound audit reader globs" is met this way rather than by a general bound, so the reading is recorded, not hidden.
  The entry describes the gap without prescribing a general bound, so this is not a scope deviation.
- Duplicate carve-out: D3 keeps `fabricvocab.BareTokenInRootDoc` and drops the test-local `ReplaceAll`, with an acceptance criterion that the scan admits the same text, as the entry's "no change in what the scan admits" requires.

No other board entry covers `tools/tokencount`, the read-only classifier or the vocabulary scan.

## Overlap with runs in flight

`lyx loom peers` lists no landed parent commits.

- `convergence-plan` (Publish) changes the Go package `internal/fabricengine`, which this run also changes.
  Its changes are `archive.go`, `archive_integration_test.go`, a new `archivetag_test.go` with top-level test `TestArchiveTagSlug`, `cleanupremotewarp.go`, `remoteleftover.go`, `trailer.go` and `trailer_test.go`: an `ArchiveTagPrefix` constant, `ArchiveTagSlug` and `TaskCommitOf`.
  This run changes `refscanner.go`, `refscanner_test.go` and `doc.go` in that package.
  The files are disjoint and the concerns unrelated, but a Go package both runs change is an overlap this check escalates.
  Its decision record names `tools/tokencount` as prior art it does not change, and lists `tokencount-cutoff-skew` as a risk to its own week bucketing; neither contradicts D1.
  Both runs touch `cmd/lyx/testdata/test-budget.yaml` only if this run adds a top-level test, which its constraints rule out; that file is bookkeeping and does not escalate on its own.
- `operator-followups` (Webster): its plan names no file in `internal/fabricengine`, `internal/lyxcwd`, `internal/fabricvocab` or `tools/tokencount`.
- `intake-bugs-1012` (Plan-Write): its decision record touches `internal/lyxtemp`, `internal/planparser`, `internal/gatecli`, `internal/shedverbs`, `internal/batcher`, `internal/loomcli` and `internal/state`, none of which this run changes.

No PATTERN entry is changed by both runs.

## Verdict

The design passes the scope check.
It is escalated only for the `internal/fabricengine` package overlap with `convergence-plan`, for the parent to confirm the two runs may land independently.
