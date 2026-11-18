---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed unwrapped return from Deps.landingSummary cannot tell the parse error from the engines and regression-check errors the same function returns, so Finalize could not word only the parse error as parse change description. Instead landingSummary became Deps.completeSummary, which takes the parsed summary and returns the credit block; each caller (Finalize.Call, PRReview.pullRequestText) calls summaryparser.Parse itself and wraps that error with its own wording, as before."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

## What changed

- F1: `internal/shuttleparity/seats.go` declares `soloSessionHolder`, and the three catalog entries use it.
- F2: `Finalize.landUnderLock` reads `parentHoldsWork` for `ownerChecks` and again after it returns nil, and passes the second answer to `landCheck` and `syncParent`.
  Both failed reads halt through `ownerHoldsReadReason`, one text.
  The summary's helper bullet and behaviour note, and the `doc.go` sentence, now name the second read.
- F3: see the differently-fixed note above.
  Finalize still words a parse failure as `parse change description`, and PR-Review as `read the change description`.
  The matching behaviour note is gone from `_lym/landing/summary.md`.
- F4: `ParentAlign.readAttempts` reads `RowEntry` first, ahead of `moveScratchAttempts`.
  `readRowAttempts` takes the entry as a parameter, and `Finalize.readAttempts` resolves it after the scratch move, so Finalize keeps its order.
  The matching behaviour note is gone from the summary.
- F5: `prreview_test.go` gains `OptionalSeamsLeftUnsetRunTheRow`, which leaves each name of `optionalPullRequestSeams` unset and requires `Call` to succeed.
- F6: `go run ./tools/codestats -root internal/landingshed` on the final tree gives production code 5,146, production comments 2,574 and test code 11,014.
  The summary's table, saving, percentage and comment and test sentences use those figures.

The gate `lym gate test ./internal/landingshed ./internal/shuttleparity` passed after the code edits (923 passed, 0 failed).
The final commit changed only a package doc comment, and `go build` passed after it.

A gate then flagged comment lines this round added that wrapped mid-sentence: in `doc.go`, in `landUnderLock` and in the `completeSummary` doc.
Each is now broken at sentence boundaries, one sentence per line; the doc.go sentence was split into three, and no code changed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: the behaviour note still said the parent-holds read happens once; fixed as proposed in `_lym/landing/summary.md`.
- fix-consistency, LOW: the helper bullet still named a shared description read after F3's split; fixed as proposed in `_lym/landing/summary.md`.
- fix-proposal, LOW: the same behaviour note; duplicate of the first consistency finding, fixed with it.
- fix-proposal, LOW: `doc.go:209` did not name the second read; fixed as proposed, a sentence added.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
