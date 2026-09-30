---
verdict: APPROVED
rationale: "Card 1 carries D1-D7 and every acceptance criterion; the reviewer's four findings (weft-dirty under-assertion, narrow integration verify, incomplete ImpactSummary, lightweight reuse tag) are all fixed in the current plan, and a repo sweep found no old-order dependant or production caller the card misses."
---

The plan under review is `_lyx/plan/00-overview.md` and its single card, `01-refuse-before-archive.md`.
Round 1's report approved the plan with four findings.
The fixer applied all four, and I checked each one against the current files:

- F1 (MEDIUM) is resolved.
  `TestRemove_WeftDirtyRefusalPushesNoTagAndResumeArchivesOnce` now asserts the full acceptance criterion 1 after the refused call.
  It checks the absent reconcile text, the empty mutation record, and the portal, launchers, both worktrees and both branches.
- F2 (LOW) is resolved.
  The `## verify:` section runs `go test -tags integration ./...`, which covers the decision record's open risk and every package the card targets.
- F3 (LOW) is resolved.
  The ImpactSummary names both production callers, `fabriccli/fabric.go:892` and `battencli/wire.go:437`, which match a repo grep.
  It also states what those callers observe differently.
- F4 (NIT) is resolved.
  The planted reuse tag is lightweight, and the card gives the reason: `archiveWeftTip` compares the ref's objectname to the tip.

My own checks against the rubric:

- **Fidelity.**
  The Intent's step order matches D1 exactly, and `force` still skips only the two dirtiness checks (D4).
  The status probes refuse before the archive (D5).
  The card covers all of D6's test changes and all of D7's doc sites.
  All four Constraints hold: sentinels and base texts stay unchanged, and the archive still precedes every removal.
  The Out-of-scope files stay untouched.
- **Old-order sweep.**
  A grep across the repo for the remedy helper, the old test names and the "side-effect" anomaly wording finds only files the card targets.
  One more mention exists at `livestate_doc_test.go:155`, inside a numbered mutation-test history entry.
  The card deliberately leaves those entries as written because they record past runs, which is correct.
  `nameStrandedPortalTeardown` appears only in `remove.go`, so the verify's negative grep can hold.
- **Feasibility.**
  Every test helper the card names already exists in `fabricengine_test` under `//go:build integration`: `mustWeftRepoRoot`, `commitLyxFileInWeft`, `tagTargetAt`, `archiveTagsAt`, `branchExistsAt` and `assertExists`.
  A broken warp gitfile does not trip anything ahead of the warp status probe.
  `mergeBlocksMutation` and `refusePrimeSlug` both answer "not blocked" on error, and `mergeSourceInFlight` resolves from the hub location.
- **Granularity.**
  One card is justified: the reorder alone turns the old-order tests red, and the replacement tests share files with them.
- **ImpactSummary** states a real blast-radius conclusion.
  No `Custom` card is present.

The round-1 report has no `## Focus departures` section, so there is nothing to ratify or reject.
