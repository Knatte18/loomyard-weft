# Card 10 — stencil-verdict-handoff

**Edit:**
- `//contracts/stencils/webster/webster-template-master.md`
- `//contracts/stencils/landing/landing-template-describe.md`
- `template_test.go`

**Intent:** Hand the integration verdict to webster: Master ends `done` over a `FAILED` report instead of `stuck` (D2).

In the Master stencil's integration-suite stage, the `status: FAILED` branch changes from `outcome: stuck` to `outcome: done`.
Master still neither localizes, fixes, nor re-forks;
its `summary.md` notes that the integration suite reported `FAILED` and that webster triages it after the session ends.
Replace the paragraph promising only a SHA-bisect with one stating webster reruns the suite, compares it against the plan's starting commit, and escalates only a regression.
Leave every other `outcome: stuck` rule in the stencil untouched.

In the Describe stencil, beside the existing `## Integration suite failed` rule, add that a `## Integration suite triage` section is carried into the description as an item for the reviewer to check, naming the flaky and pre-existing identities.

In `template_test.go`, assert the rendered Master prompt's `status: FAILED` branch instructs `outcome: done` and no longer instructs `outcome: stuck` for that branch.
If `internal/landingshed`'s describe tests pin the Describe stencil's text, update them to match.

**ImpactSummary:** Changes two seeded stencils' instructions; an operator-edited stencil copy keeps its old text by the Stencil Ownership Invariant, and card 9's triage still runs over a Master `stuck`, so an old Master stencil cannot bypass it.

**Commit:** 10: stencil-verdict-handoff
