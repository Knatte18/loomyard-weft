---
verdict: APPROVED
rationale: "All four round-2 findings are fixed and verified against HEAD: card 19 now targets every newPortalLauncherTestLocation caller (hubworkspace_test.go, launcher_content_test.go, origin_test.go), card 44 targets reedengine's strand_test.go and lifecycle_test.go, card 25's MergeShuttle reproduces mergeresolve's scripted fake and card 29 converts both non-cycle-forced fakeShuttle copies, and card 22 names smoke_parentreview_test.go; no new finding."
---

Round 2's review raised two blocking findings, one medium and one nit; the report carries no `## Focus departures` section, so there is nothing to ratify.
I checked each finding as legitimate under the rubric and then checked each fix against the code at HEAD.

**F1, card 19.**
`grep -rlw newPortalLauncherTestLocation internal cmd` returns `portallauncher_test.go`, `hubworkspace_test.go`, `launcher_content_test.go` and `origin_test.go`, all now in card 19's Edit list.
The Intent converts every call and rewords `origin_test.go`'s header, and the ImpactSummary states the six-file, two-package blast radius.

**F2, card 44.**
Outside `reedcli`, `equalStringSlices` appears only in `reedengine/reconcile_test.go`, `strand_test.go` and `lifecycle_test.go`, and card 44 now edits all three.
`reedcli/watchdog_test.go`'s copy stays with card 16, which already removes it.
The four `containsPath` definitions match card 44's four other Edit targets.

**F3, cards 25 and 29.**
The finding is legitimate fidelity to the acceptance criterion that no consumer package keeps its own merge-shuttle fake outside D1's cycle-forced cases.
`go list -deps` over `shuttleengine`, `burlerengine`, `shedengine` and `websterengine` reaches neither `mergeresolve` nor `frictionengine`, so neither package is cycle-forced.
Card 25's `MergeShuttle` now matches mergeresolve's `fakeShuttle` exactly: indexed `Results`/`Errs` with a zero answer once exhausted, recorded `Specs`, and a `DuringRun(callNumber, spec)` hook called before the answer.
It also satisfies `frictionengine.Shuttle`, which has the same one-method `Run` signature as `mergeresolve.Shuttle`.
Card 29 converts both fakes and keeps every scripted value and spec assertion, and its Card Index line matches.

**F4, card 22.**
`gateRepromptReadEngine` is referenced from `smoke_gate_test.go` and `smoke_parentreview_test.go` only.
Card 22 now edits the latter, makes the value-literal constructibility of the rewritten type explicit in its Intent, and names the dependent in its ImpactSummary.

There is no rework round (`_lyx/loom/rework/` does not exist), so the decision record alone is the answer key.
The fixes add no scope beyond D1 and the acceptance criteria.
The `## verify:` section still runs untagged and `integration` tests over `./...` and vets `smoke`, and cards 22 and 29 each run the loomcli smoke tests as a card-level verify.
