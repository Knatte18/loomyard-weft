# Fixer report — webster round 1

## Changes

- F1: restored the ten listed `PATTERN.md` entries to their merge-base text, then fixed with my own change after the gate.
  The restore left `PATTERN.md` 264 bytes past the overview headroom line, so no test ran.
  The proposed full restore is not possible under the headroom, so eight entries stay shortened and two keep a restored qualifier ("orch stencils are exempt" in `PATTERN-edit-directive`, "never duplicated" in `PATTERN-testkit`).
  Each shortened entry's qualifier is carried by its `pattern/` background file, and `PATTERN.md` is now 15532 bytes, under the 15564-byte line.
- F2: `fanSetting` counts a null list entry as `""` instead of refusing it, fixed as proposed.
  The new row `NullEntryInAListCountsAsASoloFirstRound` is in the `ClusterFan` table of `entries_burlerhalves_test.go`, since the case is no longer a refusal.
- F3: reworded the `loomSegmentPerches` comment, fixed as proposed.
- F5: `opts.FixFanKey` is now `fixFanRef.Key`, fixed as proposed.
  The pinned expectations in `entries_burlerhalves_test.go` and the `shedadapters/burlerfix_test.go` fixture and expectation moved to the bare key.
- F4: see Disputed.

## Disputed

- F4, premise false: the refusals the finding calls untested are already pinned in `internal/shedrecipe/entries_bouncer_test.go:338-346` by the rows `ReferenceWithNoDot`, `ReferenceWithEmptyModule`, `ReferenceWithEmptyKey`, `FanValueNotAFanOrList`, `FanListHoldingANonString` and `AutoAcceptNotABool`, each asserting the message.
  The one missing case, a list with a null entry, is covered by the F2 row.

## Deferred

- Self-check proposal fork, LOW: with `FixFanKey` a bare key, `burlerengine/fix.go:322` still reads "remove <key> from the profile" when the fixer's model has no config key, though the key is a `loom.yaml` setting.
  Deferred: that branch is reached only by a segment with no fix key, and rewording the `burlerengine` refusals is a change the review did not ask for.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `shedadapters/burlerfix_test.go` fixture kept the old citation. Fixed as proposed.
- fix-consistency, LOW: F4 should be disputed. Applied, see Disputed.
- fix-proposal, LOW: the `fix.go:322` wording above. Deferred as stated.
- fix-proposal, LOW: bare key loses the `in loom.yaml` locator in `shedadapters/burlerfix.go:142`. Deferred with the item above, same cause.
- fix-affected-set, LOW: the same stale fixture. Fixed, duplicate of the consistency finding.
