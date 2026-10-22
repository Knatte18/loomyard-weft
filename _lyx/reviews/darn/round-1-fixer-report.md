# Fixer-report, darn round 1

## Changes

- F1 (fixed as proposed): `contracts/stencils/orch/orch-template-role.md` line 161 now says the verb alone records a darn run's decision once Darn-Write has committed its record, with no message of the orch's own.
  Line 160 is unchanged.
- F2 (fixed as proposed): `internal/loomcli/decision.go` now routes the claim choice through `postDecisionNotice` over injectable `noticeClients`, and `TestPostDecisionNotice_SignsByCaller` in `decision_test.go` covers the callers `hub:orch`, empty, unparsable and `hub:task-a:arbiter`.
  The review proposed an `export_test.go` seam; the test is in `package loomcli`, like the existing decision tests, so none is needed.
- F3 (fixed as proposed): `postDecisionNotice` posts once more as the program when the strand post fails with `creelengine.ErrStrandClaimOutsidePane`; other failures are not retried.
  The test has rows for the fallback and for a non-retried failure.
- F4 (fixed as proposed): the dropped sentence about the darn bound and the clause "the bullet is checked with the record" are restored in the `decision add` Long text.
  A long-help cap does exist: `maxLongBytes` (2000) in `cmd/lyx/clitree_test.go`, which I had missed by searching `internal/` only.
  The restored text put `decision add` at 2132 bytes and failed `TestCLIForm_EveryCommand` in the verify gate.
  Following F4's rule, I trimmed only the notice paragraph, which now reads that the run's other live agents are notified and the envelope lists "notified" and "notice_errors".

Verified with `lyx gate test ./cmd/lyx ./internal/loomcli` (tier 0, green) after the gate repair.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
- fix-consistency: LOW, line 161 does not say a later producer reads the record when no agent is live; fixed as proposed by adding that clause to line 161.

## Disputed

None.

## Deferred

None.
