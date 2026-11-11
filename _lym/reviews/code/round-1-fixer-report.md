# Fixer report, round 1

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  `internal/shuttlecli/cli_test.go` gains `TestRunCLI_AwaitForksResolvesFromTheRecordsSibling`.
  It runs `await-forks` from the records sibling's `_lym/plan` directory and expects it to pass the resolution gate, reaching "no shuttle strand".
  It runs `state` from the same directory and expects the outside-anchor error.
  Commit `0a96f179`.
- **F2 (LOW), fixed as proposed.**
  `internal/lymcwd/lymcwd_test.go` gains `TestResolveWithin_NoOwnerOrNoMarker`.
  Its rows are a checkout named with the `-weft` suffix and no owner beside it, which resolves as itself, and a checkout with no marker resolved from a subdirectory, which gets AnchorRel ".".
  Commit `dc680a34`.
- Both file headers now name the new coverage, per the self-check's consistency and affected-set findings.
- Gate repair: three comment lines of 120 characters or more were split at sentence boundaries.
  They are the `lymcwd_test.go` header line, the `TestResolveWithin_NoOwnerOrNoMarker` doc line and the `cli_test.go` doc line of the F1 test.
- Third gate repair: the F1 test moved out of `cli_test.go` into `internal/shuttlecli/resolve_verbs_test.go`, a tier-1 file of its own.
  The verify gate refused `gitkit.CopyRepo` and `weftname` in `shuttlecli`, and a tier-0 file may not hold tier-1 tests.
  The test now builds its hub with `hubforge.NewHub` at a subpath anchor plus one pair, and runs from a `plan` subdirectory of the pair's records sibling.
  The `cli_test.go` header line and the `gitkit` and `weftname` imports added for F1 are gone.
  The `slackengine` relay failure in the same verify run is in a package this round did not touch, and is left as it is.
- Second gate repair: the `TestResolveWithin_NoOwnerOrNoMarker` doc had been split mid-sentence.
  It is now two whole sentences, one per line.

Gate: `lym gate test ./internal/lymcwd ./internal/shuttlecli --tier 1` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, stale `cli_test.go` header; fixed as proposed.
- fix-consistency: LOW, stale `lymcwd_test.go` header; fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: LOW, stale `lymcwd_test.go` header; duplicate of the consistency finding, fixed as proposed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
