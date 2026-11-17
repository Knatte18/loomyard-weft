```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with its sibling cards, decisions and cross-references (fix-consistency lens).

**LOW: card 14's reason for tender resume is now stale**
- Changed text: card 15's new sentence, "`resumeAside` gains `Hold`, so the verdict `hubRunIntents` judges for `lym tender resume` sets the hold aside".
- Problem: card 14 still says the `Verdict` seam of `hubRunIntents` reads through `ReadRunPulled` "so a hold another machine wrote is read (D5)". That verdict now ignores `Held`. A tender resume reads the hold through card 15's `HubDeps.BoltHold` instead, which runs its own `PullBolt`. Card 14's stated purpose for that caller no longer holds, and the resume now pulls Bolt twice.
- Evidence: card 14 Intent, "`loomcli`'s `hubRunVerdict` … and the `Verdict` seam of `tendercli`'s `hubRunIntents` (behind `lym tender resume &lt;slug&gt;`) read through it", under the goal "so a hold another machine wrote is read". Card 15 line 48 sets the hold aside, and card 15's `HubDeps` "fills `BoltHold` … after a `boardengine.PullBolt`".

**LOW: card 13's ImpactSummary overstates who now waits under a hold**
- Changed text: the same card 15 `resumeAside` change.
- Problem: card 13's ImpactSummary still says "every act-rule caller now waits under a hold". After card 15, tender resume's act-rule caller sets the hold aside and does not wait.
- Evidence: card 13 ImpactSummary, "so every act-rule caller now waits under a hold".

**LOW: card 4's ImpactSummary leaves out the new exported reader**
- Changed text: card 4's new `IdentityRefusal` and `identityRefusal` Create targets.
- Problem: the ImpactSummary still reads "New setter and identity writer only". The card now also adds an exported read-only refusal reader, and card 8's `dobbyStatus` calls it.
- Evidence: card 4 lines 12–13 and 52; card 8 Uses `plan:internal/boardengine#IdentityRefusal`.

Tool calls:
- One Bash call read cards 12, 14 and 22 in full and grepped the plan for `refuseStartOwnedElsewhere`, `TestRefuseStartOwnedElsewhere`, `BoltHoldRequest`, `resumeAside`, `beamcli/doc.go`, `statusResult`, `machine_refused` and `IdentityRefusal`. My context did not hold those sibling cards or these cross-references.
