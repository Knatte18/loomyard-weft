---
verdict: APPROVED
---

Alignment at `pr-gate` for `landing-trailer-text`, measuring pull request 717 at `888757b4025fee227fc33acdebb69900742f4a81` against `main` (merge base `743c11f0d`).

The decision record holds no `Added after Discussion` entries, so the measure is the board entry's body and its operator decision of 2026-10-17.

## Scope against the board entry

- `co_authored_by` is gone from `landing.yaml`; `trailer_text` and `engine_trailers` replace it in `internal/landingshed/config.go` and `internal/landingshed/template.yaml`.
- `Config.TrailerBlock` copies both values verbatim, with no prefix, and `summaryparser.LandingMessage` appends the block after a blank line, or nothing when the block is empty.
  This covers the multi-line fixed text, an empty `trailer_text`, an engine left out or mapped to empty text, and the no-trailer-block case.
- An engine's line lands only when that engine ran a session: shuttle's Start records the engine it resolves for each run in `engines-run.json`, `Runner.EnginesRun` reads it with the surviving run records, and Finalize reads it through the told `Deps.EnginesRun`, wired in `internal/loomcli/landingdeps.go`.
- The template default holds `trailer_text: ""` and the `claude` line `Co-Authored-By: Claude <noreply@anthropic.com>`.
- `landingshed.MigrateConfig`, registered in `internal/configreg/configreg.go`, moves `co_authored_by: <value>` to `engine_trailers.claude` as `Co-Authored-By: <value>`.
- The Describe validation is untouched, and the describe stencils still forbid a `Co-Authored-By` line.
- `internal/landingshed/doc.go`, `contracts/specs/loom-final-summary-spec.md`, both describe stencils and `docs/overview.md` are updated, and no other file in the tree names `co_authored_by`.

The extra choices the record makes beyond the body all serve its points and are not scope additions:
the unknown-engine refusal in `LoadConfig` guards the per-engine map,
and the ledger is how "read from the run's own records of the sessions it spawned" survives the orphan sweep.

## Open risks carried over

- Darn-Review round 1 carried `start-ledger-write-failure-untested` open.
  Commit `bc012d593` adds the test that a failed ledger write fails Start, removes the run directory and registers no strand.
  It names no parent correction, so it does not escalate.
- The record's own risk, that a run in flight across the binary change may miss an engine whose directories were swept before the ledger existed, is a transition bound the record states, not a scope gap.

## Overlap

- Peer `creel-cli-output-fixes` (row 14, own row 22) changes only `internal/creelcli`, `internal/creelengine` and `tools/sandbox/SANDBOX-CREEL-SUITE.md`.
  No file, behaviour or PATTERN entry overlaps this run, so no landing order applies.
- `lyx loom peers` lists no parent commits.
- Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml` (shuttleengine and landingshed rows) and the landing paragraph in `docs/overview.md`; neither escalates.

No board entry is made obsolete by this run.
