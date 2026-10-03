---
verdict: APPROVED
rationale: "Both round-1 open findings and all six round-2 findings are fixed in the 20-card plan: card 15 now edits gitwrap.go and adds commitsSince/fixCommitRejection there, so verifygate.go never calls gitrepo.New; card 17 deletes integrationFixHeadRefusal and commitParentCount (whose only caller outside gitwrap.go is integrationfix.go), rewords rawgitmutation_test.go's FabricBisector comment, drops merge_verbs.go's bisect clause with its test assertion, and fixes the three stale comments; new card 20 retires CheckoutDetached/RestoreBranch, licensed by D4, and covers every file that names them"
---

# Plan-Review round 2 — judge verdict

## Round-2 findings: all resolved

I checked each fix against the current plan and the repository at `5805c58fb`, not against the fixer report.

- **F1 (MEDIUM), commit walk in `verifygate.go`:** card 15 now lists `internal/websterengine/gitwrap.go` under `**Edit:**` and no longer under `**Uses:**`.
  Its commit-walk section adds `commitsSince` over `commitsBetween` and `fixCommitRejection` over `CommitParents`, `resolveParentTips` and `parentMergeRejection`.
  `verifygate.go` reaches both through a seam.
  `fixCommitRejection` resolves the tips lazily on the first merge, and a tip-resolution error rejects that merge.
  That matches `reconcileHead` (`gitwrap.go:86`), so a nil `ParentBranch` accepts no merge.
  The integration test drives the real helper, and `TestNoRawGitMutation_WebsterProductionSource`'s allowlist already covers `gitwrap.go`.
- **F2, gitwrap leftovers:** card 17 targets `gitwrap.go` and deletes `integrationFixHeadRefusal` (`gitwrap.go:83`) and `commitParentCount` (`gitwrap.go:240`).
  Outside `gitwrap.go`, `integrationfix.go:379,397` are their only callers.
  `commitsBetween` stays under `commitsSince`, and `commitChangedPaths` stays under the card hint.
- **F3, raw-git guard comment:** card 17 targets `cmd/lyx/rawgitmutation_test.go`, rewords the line-10 `FabricBisector`/`FabricResetter` example, and adds the fix-commit walk to the `gitwrap.go` allowlist reason.
- **F4, merge-in bisect text:** card 17 targets `internal/fabriccli/merge_verbs.go`, whose lines 32 and 142–143 hold the bisect clause.
  It also targets `mergein_webster_integration_test.go`, whose line 73 asserts `pre-merge trees`, as D2 requires.
  No other file asserts either string.
- **F5, stale comments:** `verifyrun/run.go:3`, `recoverbatch.go:460` and `state.go:255` are each targeted and named in card 17's Intent.
  The other surviving Go mentions sit in files card 17 already edits or deletes (`webstercli/wiring.go`, `webstercli/cli.go`, `websterengine/doc.go`, `summary.go`, `triage.go`).
  The doc and spec mentions are card 19's.
  Merriam's stencil line 134 sits inside the integration-suite section card 16 removes.
- **F6, detached-checkout primitives:** card 20 is licensed by D4 ("every detached checkout of the live worktree go").
  Its targets cover every file that names `CheckoutDetached` or `RestoreBranch` outside card 17's own files:
  `gitrepo.go`, `gitrepo_test.go`, `parity_test.go:345`, `gitrepo/doc.go` (lines 27, 69, 134, 159, 255–256), `warpforward.go`, `warpforward_integration_test.go`, `fabric.go:14`, `fabricengine/doc.go` (1105–1106, 1270–1271) and `gitrepoboundary_test.go:67–68`.
  The remaining test fakes are in `triage_test.go` (card 17) and `integration_test.go` (deleted by card 17).
  `ErrInvalidSHA` keeps live callers in `ancestry.go` and `push.go`, and `CurrentBranch` stays.
  Its `**Uses:**` of `triage.go` orders it after card 17, which edits that file.
  It removes code and opens no surface, and its ImpactSummary states its reach.

F6 came from job B, outside round 2's focus entries, and the review has no `## Focus departures` section.
I ratify it on the merits: it is a fidelity gap against D4's retiring list.

## Round-1 carry-forward

The two entries round 1 left open are F1 and F2 above.
Round 1's other fixes still hold: card 3 still targets the driver-bootstrap test, the 14/15/16 split and both exported gate constructors are intact, and `accumulatedCardSHAs` survives card 17.

## Checked and clean

- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover every targeted package, including card 20's `gitrepo`, `fabricengine` and `cmd/lyx`.
- No card is `Custom`, and card 19's `Prosa` targets are all files.
- Fidelity: D1–D16 and every Constraint are still carried by some card, and card 20 is the only new scope; D4 licenses it.
- Attack surface: card 20 and card 17's guard-comment rewording loosen no guard.
  Card 15's new `gitrepo.New` use stays inside the already-allowlisted `gitwrap.go`, and its queries are read-only.

## Observation, not a finding

Card 14 `**Uses:**` `gitwrap.go` and `state.go`, which cards 15 and 17 edit.
With card 15's new `gitwrap.go` target and the shared-target edges, `SequenceBatches` condenses cards 14–17 into one cycle.
A cycle runs in declared order, which here is the correct order, so execution is unaffected and Merriam only logs a cycle warning.
The rubric's dependency-edge exclusion covers this, so I do not raise it.
