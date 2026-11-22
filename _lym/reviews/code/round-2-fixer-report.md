---
dispositions:
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed-differently
    reason: "The reviewer's flag text ended with removing harden-live-copies with lym config reconcile, which cannot work: applyReinterpretations writes the board marker after a flagging Rewrite too, and skips a reinterpretation whose marker exists (internal/hubreconcile/reinterpret.go), so reconcile never reruns the rewrite and the per-name gates map keeps an operator-named entry. The flag and the landing summary tell the operator to delete the key by hand instead."
---

## Changes

- **F12** (commit 4ca9f7f4):
  - Deleted `focusFileFormat` from `internal/hardenshed/campaign_rows.go`.
  - `Campaign.RoundFacts` now names the charter through `shedadapters.FocusPath(runDir, round)`.
  - Renamed `focusPath` to `FocusPath` in `internal/shedadapters/round.go` and at every caller and test in `internal/shedadapters`.
  - Updated the comment in `internal/loomrecipe/fixture_test.go` that names it.
  - `TestCampaignRoundFactsNameTheCharterAndTheOperatorsAsk` keeps its own literal.
- **F13** (commits 21007036 and the follow-up wording commit):
  - Added `gateslot.RetireHardenLiveCopies` and `RetiredCopiesReinterpretationID` in `internal/gateslot/reinterpret.go`.
  - Registered them as a `configreg.Reinterpretation` on the gate module in `internal/configreg/configreg.go`.
  - The rewrite removes `gates.harden-live-copies` when `harden-live` (the template's tier 0 when absent) names exactly one tier below `BilledTier` and equals it.
  - Otherwise it leaves the file and flags the way forward: set `harden-live` with `lym config gate --set`, then delete the retired key by hand.
  - Added `TestRetireHardenLiveCopies` in `internal/gateslot/reinterpret_test.go`: removal, absent live key, several tiers, the billed tier and no copies key.
  - Updated `_lym/landing/summary.md`:
    - the design-decision bullet now says the key is folded into `harden-live`;
    - the "No hub gate writes" bullet says two harden keys;
    - Merge danger is one-way and names every hub's `gate.yaml`.
    - That file is not committed, since `_lym` is not mine to commit.

- **Gate finding** (comment line length): rewrote the doc comment of `TestRetireHardenLiveCopies` in `internal/gateslot/reinterpret_test.go` as one short sentence per line, since a split at a comma was flagged as a wrapped break.

- **Gate finding** (verify, `TestTierTags_OneTierPerTestFile`): `TestRetireHardenLiveCopies` now opens with `tiergate.Require(t, 0)`, as `PATTERN-test-speed` requires of every test.

## Self-check

- fix-code-correctness, MEDIUM: the flagged way forward cannot be carried out, so it is fixed differently with the F13 change above.
- fix-code-correctness, LOW: re-encoding may drop blank lines of a hub's gate.yaml. Comments and values survive, so I left it as is.
- fix-proposal, MEDIUM: duplicate of the first finding, merged into it.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
