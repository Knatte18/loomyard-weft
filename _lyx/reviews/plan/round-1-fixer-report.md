# Fixer report — plan round 1

## Changes

- F1, BLOCKING, fixed as proposed: in card 5's Intent, the approve and reject `runStart` refusal's way forward is now `transient, re-run lyx loom approve` (and `reject`).
  The card states that `loomCLI.resolveRecipe` (`internal/loomcli/arm.go:127-133`) refuses an unreadable or undecodable self seed in the persistent pre-run (`cli.go:230-252`, `arm.go:184`) before the verb body runs.
  So the verbs' own branch is a propagated read failure that `refusal-spec.md` exempts as transient, and D1's refusal is carried upstream.
  The `SeedUnreadable` rows now want "read the run's start" and "transient, re-run lyx loom approve" (and `reject`).
  The ImpactSummary now names the new refusal as the verbs' transient read of the run's start.
- F2, MEDIUM, fixed as proposed: card 5's Intent says that for the self run, `resolveRecipe` refuses an undecodable seed on every armed verb, and `sharedbootstrap.go:100` refuses it on the start path, both before `wireLanding`.
  `wireLanding` still returns `seedRunStart`'s error unwrapped, as a propagated read failure.
  The `TestWireLanding_TellsLanguageSkills` bullet keeps only the stamped-seed `RunStart` assertion.
- F3, LOW, fixed as proposed: card 5 now has `internal/landingshed/doc.go`'s Publish paragraph (`doc.go:6`) gain one sentence on the run-start bound and the zero-start fallback.
  The `FindPullRequest` doc-comment instruction stays.
- F4, LOW, fixed as proposed: card 5 has `runstart.go`'s file header gain the run-bounded pull-request lookup beside the seed's start read.

## Self-check

- fix-consistency, LOW, `_lyx/plan/lens-review.md`'s test-economy dispute still cited the removed decode-error assertion: fixed as proposed.
  A note under that bullet says round 1 (F2) dropped the assertion, so the dispute now rests on the `Deps.RunStart` assertion alone.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set, LOW, "matching the sibling read refusals" read as covering reject, but `rejectVerb`'s sibling refusals (`internal/loomcli/reject.go:90-98`) carry no way-forward clause: fixed as proposed.
  The card now says the approve message matches `approveVerb`'s siblings, and the reject message carries the clause though its siblings carry none.

## Disputed

None.

## Deferred

None.
