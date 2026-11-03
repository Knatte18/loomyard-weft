# Fixer report, round 1

F2 and F4 are `decision`-class findings and went to the parent, so this report leaves them out.

## Changes

- **F1**, fixed as proposed, in `cd6a1811` and `a1d9865f`.
  - `internal/testkit/envkit/envkit.go`: `Landing.DriftVerdict` is out of `nilLegal`, and `LandingDeps` fills it with a stub returning an empty section and nil.
  - `internal/loomcli/wiring_seams_test.go`: `Landing.DriftVerdict` is in `intentionallyNil`, with the reason `Landing.EnginesRun` carries.
  - `internal/loomcli/landingdeps_test.go`: `TestLandingDeps_DriftVerdict_RendersTheScopeRecord` asserts the seam answers no section before a scope record exists and the rendered section once one does.
  - `internal/loomcli/wiring_publish_integration_test.go`: `TestWire_DescribeTableCarriesTheDriftVerdict` asserts the `DescribeTable` closure puts the rendered section in the table's `drift_verdict` value, over a real hub and pair.
  - `internal/battencli/wire_seams_test.go`: `Landing.DriftVerdict` is in batten's own seam list, a twin the review did not name and the self-check found (see Self-check).
- **F3**, fixed as proposed, in `418d037d`.
  The Finalize pull-request test has a row with `Deps.DriftVerdict` set, asserting the fake API's merge message holds the body, the section, then the credit block.
- **F5**, fixed as proposed, in `418d037d`.
  - `TestPRReview_Call` has a refresh row with the verdict set, asserting the title and body update carries the section once.
  - `TestWithSection` pins the empty-section, trimmed-body and whitespace-only-body cases.
    The review asked for the whitespace-only case as a description row; a direct table over the helper asserts it without the credit block diluting the body.
  - `TestPRReview_Session` has a row whose `DriftVerdict` seam succeeds on the body's read and errors on the spec's read, asserting an error and no session start.
- **F6**, fixed as proposed, in `27b553d2`.
  `requireCompareRecorded` takes the baseline and compared commits separately, and the passing-compare, agent-Done and final-failing rows capture at `base` against a HEAD of `aaa`.

F3 and F5 share one commit because both change `internal/landingshed/finalize_pr_test.go`.

Gates: `lym gate test` over `./internal/driftengine`, `./internal/landingshed`, `./internal/loomcli` and `./internal/testkit/envkit` at tier 0, `--tier 1 ./internal/loomcli`, and `./internal/battencli` at tier 0, all with no failures.

## Self-check

- fix-consistency, BLOCKING, battencli's `intentionallyNil` lacked `Landing.DriftVerdict`: fixed as proposed in `a1d9865f`.
- fix-affected-set, MEDIUM, the same battencli finding: merged with the one above.
- fix-code-correctness: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
