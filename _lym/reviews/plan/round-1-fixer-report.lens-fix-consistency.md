```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checked whether the fixer's edits to cards 1–3 still agree with their sibling cards, the overview, the decision record and the code sites they name.

- LOW — card 1, `**ImpactSummary:**` (not changed, but the F4 fix reaches it).
  Card 1 now edits `internal/loomcli#loomRerouteCapWayForward` and its two tests, which are new Edit entries.
  The ImpactSummary still describes the card's reach in `internal/loomcli` as only the two removed `Deps` fields and the overlap note.
  Evidence: card 1 line 57 says "`loomRerouteCapWayForward` drops its clause for a parent-branch reroute…", while the ImpactSummary reads "the two removed `Deps` fields and the overlap note reach only `internal/loomcli`'s landing and darn wiring, `envkit.LandingDeps` and their tests, with no exported signature change elsewhere."
  The summary never mentions the change to the way-forward constant.

Nothing else disagrees:
- The changed text has no twin in `_lym/plan/00-overview.md` or `_lym/discussion/decision-record.md`.
- The `parent branch has settled` text appears only at `internal/loomcli/wiring.go:955` and `wiring_test.go:1052,1099`, and card 1's edits cover all three.
- Card 2's pawl way forward now matches decision-record.md:55 word for word.
- Card 3's Bound sentence on an arbiter that is off agrees with decision-record.md:67 after its row was deleted.
- Card 1's new HandsBack-paragraph sentence and card 2's HandsBack edit touch the same paragraph without conflict.

Tool calls:
1. grep of the plan cards and the decision record for twins of the changed phrases: my context did not show the overview or the decision record beyond line 55.
2. grep of the repo outside `_lym` for `parent branch has settled` and `parent-branch reroute`, plus card 2 and the fixer-report spec for over-cap wording: to find sites of the dropped clause that my context lacked.
3. grep of `00-overview.md` for pawl, reroute-cap, way-forward and spawn-cap text: I had not read the overview.
