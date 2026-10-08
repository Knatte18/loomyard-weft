---
round: 1
focus:
  - "Caller coverage walk (`internal/planglyph/callercoverage.go`): settle whether the reference scan matches card 11's rules for package-level versus method subjects, skips `testdata`, `vendor`, `.`- and `_`-prefixed directories, ignores comments and string literals, and excludes a subject's own resolved spans."
  - "Plan-gate-only placement: settle whether `redundant-file-target`, `resign-interface-method` and `caller-uncovered` run only from `planGatePass` and never from `ValidateDispatch`, while `redundant-package-target`, `resign-not-member` and `resign-head-mismatch` run at every gate, as the check-ID decision states."
  - "Re-sign head matching in `CanonicalizeHandles`: settle whether re-sign answers are compared and discarded, never reaching `canonicalOwners`, the collision check or `RewriteRefs`, and whether a pointer-receiver change matches while a changed receiver type does not."
  - "Retirements: settle whether removing `containment-unit-overlap`, `containment-file-overlap` and `handle-unreferenced` leaves no dangling reference in code, `doc.go`, the spec's renumbered rows, or tests."
  - "Spec row renumbering in `contracts/specs/loom-plan-spec.md`: settle whether every `row N` cross-reference was fixed after each insertion or removal, and whether the Delete and Edit mechanical-check entries name the new checks."
  - "Per-card mechanical checks: for each card, settle whether every group's type-specific check ran against that group's own targets (Rename grep and AST, Delete `assert-no-callers`), not just the first target."
  - "Test economy in `glyphchain_test.go` and the integration file: settle whether each new row covers behavior no existing row covers, and whether each added check has a test for its blocking and clean outcomes."
  - "Comment conventions on new or changed doc comments and `doc.go` bullets: settle compliance with the repository's own constraints document, with no inferred line width."
  - "Stencil and rubric (card 12): settle whether `claims_test.go` pins the new text and the removed `handle-unreferenced` text, and whether the stencil points at spec check IDs instead of restating them."
---

Seed focus only; each entry names where to look and what to settle, and none pre-states a verdict.
