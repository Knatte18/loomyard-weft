# Fixer report — discussion round 2

## Fixed

- **F1** (MEDIUM, decision): confirmed in `internal/websterengine/recordbatch.go`: the `len(correctness) > 0` branch passes the raw `r.HeadSHA` to `failOnCorrectness`, and `failFromFindings` writes it as the failed digest's `HeadSHA`.
  D1 now says record-batch resolves the parsed report's `head_sha` once, before its first use, so the failed digest records the resolved SHA.
  It also picks the order: a correctness finding wins over an unresolved value, and the batch fails with an empty digest head, never the raw value and never D1's refusal, matching today's handling of a report that does not parse.
  The Rationale states why: a bad `head_sha` never holds off a batch failure.
  A new acceptance criterion covers both cases.
  The support log gains the rejected alternative (refusal wins) and Q14.
- **F2** (MEDIUM, design): confirmed that `readRunStartBase` (`tools/tokencount/calibrate.go`) reconstructs the raw `PATTERN.md`, while `MerriamBase` (`internal/websterengine/merriambase.go`) counts `CLAUDE.local.md` and the orchestrator directive from `pattern.Directive`, which fills the whole `PATTERN.md` into `pattern-directive-orchestrator.md`.
  D4 now subtracts exactly the texts `MerriamBase` counts except `00-overview.md`, with the directive rendered at the base commit from that commit's `PATTERN.md` and `contracts/stencils/pattern/pattern-directive-orchestrator.md`.
  `readRunStartBase` changes to the same directive, so the start fit and the new section reconstruct the same texts, and Scope names that change.
  `CLAUDE.local.md` cannot be reconstructed at a base commit: D4 assumes none, as lyx-created worktrees have none, and states the error bound.
  The fixture acceptance criterion now pins the subtracted line count to `CLAUDE.md`, the Master template and the rendered directive.
  The support log gains two rejected alternatives and Q15.

`lyx loom validate-discussion` passes after the edits.

## Disputed

None.

## Deferred

None.
