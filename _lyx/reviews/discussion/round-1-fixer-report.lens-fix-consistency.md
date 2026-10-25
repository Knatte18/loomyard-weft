```yaml
lens: fix-consistency
outcome: reported
```

Restating the task: check the fix's change set for agreement with its sibling decisions, criteria and cross-references, and look for twins left unchanged.

- **MEDIUM**: D7 recovery bullets "`A`: the move or integration did not land" and "`B`: the move or integration landed".
  - Summary: when an integrating `--set` runs on a checkout already at `B`, the two branches of the recovery cannot be told apart, and the first one destroys data.
  - Evidence: D7 says an integration runs whenever "the weft's fetched upstream has commits the weft's tip lacks and the two tips carry different markers". This includes the case D10's way forward names, "the local tip's marker otherwise", where the record's `A` equals `B`.
  - In that case the marker committed at `HEAD` reads both `A` and `B`, so the re-run cannot tell whether the integration landed.
  - If it takes the first bullet, it "deletes what `HEAD` does not track under `B/<names>`", and here `B/<names>` is the live wired content at the current anchor.
  - It also "renames `B/.lyx` back to `A/.lyx`", which is the same directory.
  - The record holds the kind of run, but the recovery tells its states apart only by the committed marker, which does not decide this case.
- **LOW**: D10's new pull and push stop rule, and the D2 empty-marker rule together with its acceptance criterion.
  - Summary: the fix made the weft push act on a marker (D10), but the empty-marker refusal list was not extended to cover it.
  - Evidence: D2 says "Every `fabricengine` path that acts on a marker, D5 to D10, refuses an own marker that is present but empty". The criterion still reads "refused by checkout, pull, `Add` from it, the anchor verb and reconcile", with no push. D10 now says a push "stops that pair … when it would change the checked-out marker".
- **LOW**: D10 rationale, which the fix left unchanged.
  - Summary: the rationale no longer covers the integration the fix added to D10.
  - Evidence: the rationale reads "each refuses before changing anything, so no half-rewired pair needs a recovery of its own". The added integration does change the checked-out marker. It is recovered only because it runs inside D7, and the rationale does not say so.
- **LOW**: support log, Turn 4 entries left beside the one Turn 2 line the fix renumbered.
  - Summary: the fix updated "D7 step 5" to "D7 step 6" in Turn 2 but left the Turn 4 lines that the new D7 contradicts.
  - Evidence: Turn 4 still says "D7 step 1 ends the dobby process alone first, then checks", "the change commit carries `Lyx-Anchor-From`", "step 3 closes the sink and step 4 reopens it" and "step 7 restarts exactly the dobbies step 1 ended". None of these is marked as revised in round 1.

No further tool calls were made.
