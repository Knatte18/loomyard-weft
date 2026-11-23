---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposed `parentReviewPRAsker(target, strandClaim)` with `preRun` passing `os.Getenv(...)` left `preRun`'s argument unpinned, as two self-check lenses showed. The ask seam now takes the claim as an argument and `parentReviewAskVerb` passes `deps.strandName`, which its own session check ties to the recorded PR-Review strand. `TestParentReviewAskVerb` asserts the seam receives the session's strand, and the integration test drives `parentReviewPRAskFunc(target)` with each claim. The function is named `parentReviewPRAskFunc` because `parentReviewPRAsker` already names a test helper in `parentreview_test.go:40`."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

# Round 1 fixer report

## Changes

- F1 (`internal/loomcli/parentreview.go`, `parentreview_test.go`, `prreviewask_integration_test.go`):
  `askPR` takes `(rec, head, strandClaim)`, and `parentReviewAskVerb` passes `deps.strandName`.
  `preRun` builds the seam with `parentReviewPRAskFunc(target)`.
  `TestParentReviewAskVerb` asserts every successful ask got the session's strand as its claim.
  The integration test drives the seam, with its deadline checked in the window around the call.
- F3 (`landing-block-pr-review-autonomous.md`, the `ask` verb's Long text): "refused or failed to deliver", reason "names why".
- F2 (`_lym/landing/summary.md`, untracked, not committed): `## Merge danger` names the new `undelivered` state in `.lym/parent-review/asks.json`, what an older binary does with it, and a widened blast radius.
  It keeps `two-way` with a justification beside it.
  The summary's stale lines (the stencil's wording, the test bullet) now agree with the code.

Commits: F1 and F3 share `parentreview.go` and form one commit, and a second F1 commit follows the self-check.
Both gates passed: `lym gate test ./internal/loomcli` at tier 0 and tier 1.

## Gate repair

The gate's reach report named code no mapped test reached: `prReviewAsker` and `parentReviewPRAskFunc` in `internal/loomcli`, and `PostAsCaller`, `notifyClientsOf` and `postNotice` in `internal/orchcli`.
Only tier-1 tests reached them.
A commit adds tier-0 tests for each:
- `TestParentReviewPRAskFunc_ReturnsTheErrorOfAHubItCannotResolveTheParentIn` drives the seam over a temp hub, reaching `prReviewAsker`.
- `TestPostAsCaller_BuildsTheClientOfTheIdentityItSignsAs` and `TestPostNotice_PostsOnlyWhenTheOrchIsRecorded` run outside a hub, reaching `PostAsCaller`, `notifyClientsOf` and `postNotice` with the line in the error.

`lym gate test ./internal/loomcli ./internal/orchcli` passes at tier 0.

A second gate repair shortened the two test doc comments that reached 120 characters, with no change to the tests.

## Self-check

- fix-code-correctness, MEDIUM: the test did not pin `preRun`'s claim. Fixed differently, as F1 states.
- fix-consistency, LOW: the summary's "refusal" wording is stale. Fixed.
- fix-consistency, LOW: the summary's test bullet names `prReviewAsker`. Fixed.
- fix-proposal, MEDIUM (F1): same gap as the correctness lens. Merged with it and fixed differently.
- fix-proposal, LOW (F2): "the next run rewrites it" has no evidence. Fixed: the clause is cut.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
