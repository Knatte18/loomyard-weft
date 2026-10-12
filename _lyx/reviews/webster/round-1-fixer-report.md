# Fixer-report — webster round 1

## Changes

- **F1** (fixed as proposed): the five undecodable-status fixtures now use malformed JSON.
  - `internal/loomengine/seedownership_test.go`: `DecodeFailurePasses` body is `{not json`; its comment no longer calls an unknown field malformed.
  - `internal/loomcli/start_midmerge_test.go`: `unknownFieldStatusBody` renamed `malformedStatusBody` (`{not json`) with a comment saying it does not parse; the row at the former line 333 is now "clean pair with a malformed status file"; the use at line 152 follows the rename.
  - `internal/loomcli/start_driver_test.go`: the use at line 648 follows the rename.
  - `internal/loomengine/seed_test.go`: the 'unknown top-level field' row is deleted, and the `TestCheckSeed` doc names malformed JSON alone.
- **F2** (fixed as proposed): `internal/battencli/overview_test.go` slug and seed names `strict` became `future-key`, the comment says the seed decodes and is listed like any other, and the expected lines are re-padded to the longer slug.
  The `-` columns still come from the seed lacking `started_at`, so the expected values are kept.
- **F3** (fixed as proposed): space added in the `TestLoadState_UnusualFiles` doc in `internal/websterengine/state_test.go`.

- **Gate finding** (fixed): the `DecodeFailurePasses` comment I rewrote in `seedownership_test.go` was fixed-column wrapped.
  It now breaks at sentence boundaries, one sentence per line, and drops the stale word "strict" from its description of the step-1 read.
  A second gate pass flagged the "Reproduced live" sentence, which still broke mid-sentence at a comma before a participle; it is now one line.

`lyx gate test ./internal/loomengine ./internal/loomcli ./internal/battencli ./internal/websterengine` is green.
One commit per finding.

## Disputed

None.

## Deferred

None.
