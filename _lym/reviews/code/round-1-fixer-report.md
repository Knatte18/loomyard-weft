# Fixer report, round 1

All eight findings are fixed as proposed.
`lym gate test ./internal/loomcli ./internal/burlerengine` passed (tier0, 0 failed).

## Changes

Commits are per file, since several findings share a test file.

- F1 (`652d2ce8`): `internal/loomcli/cli_test.go` iterates `parentReviewGroup` in `TestResolvePersistentPreRun_SkipsSlugGroups`, and the comment names the group.
- F8 (`32bd1767`): `AnswerParentAsk` in `internal/burlerengine/parentreview.go` returns the wrapped write error first, unless it is `errParentAskUnchanged`, and returns the refusal after that.
- F2, F3, F4, F6 (`e7964633`): `internal/loomcli/parentreview_test.go`:
  - F2: the findings-file row reads the written report and compares its findings (P1 MEDIUM design, P2 bare) with the file's items; `wantFindings` became `wantReport`.
  - F3: a row where the caller is the resolved parent orch answers done, and a row where the parent cannot be resolved refuses with "could not be resolved" and leaves the ask open.
  - F4: rows for an empty file (new `emptyFindingsFile` field), `[`, a missing path and a whitespace-only change, each refusing with the findings-file way forward and leaving the ask open.
  - F6: `sleepErr` field and a row refusing with "the wait was cut short" and the retry way forward; `wantErr` became a list.
- F5, F7 (`3142c0ff`): `internal/burlerengine/parentreview_test.go`:
  - F5: a `future-state` ask row in `TestParentAsks_RecordAndMoveForward` (expiry leaves it) and in `TestAnswerParentAsk` (refused as ended, state unchanged).
  - F7: the `//testtiming:keep` line above `TestAnswerParentAsk_RacingTheDeadline`.

F8 has no regression test: the review proposed none, and a failed record write is not reachable in the test fixtures.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings; it noted, without raising a finding, that the `TestParentReviewAwaitVerb` doc comment omits the cut-short refusal.
- fix-proposal: no findings.
- fix-affected-set: no findings; F2, F3, F4 and F6 have no set premise and were left unchecked.

## Disputed

None.

## Deferred

None.

## Rulings

None.
