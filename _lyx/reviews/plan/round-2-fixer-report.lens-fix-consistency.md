```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked that the changed text in cards 4, 6, 7, 12 and 16 still agrees with its sibling cards, decisions and cross-references.

- **LOW** — card 6 (`_lyx/plan/06-retire-alignment-keys.md`), **ImpactSummary:**
  - **Summary:** The ImpactSummary is now stale. The fix added a `**Prosa:**` group with `docs/overview.md` and an Intent bullet that edits the overview's loom config and reconcile paragraphs. The ImpactSummary still lists only the code readers.
  - **Evidence:** The ImpactSummary reads: "Four config keys and the alignment resolver go; the readers are loomengine and its tests, configreg's tests and three loomcli and webstercli test seeds."
  - **Change:** Add the `docs/overview.md` loom config and reconcile paragraphs to it.

Checked with no finding:
- **Card 4's `docs/overview.md` waiting-note clause:** The new sentence names a verify and a shuttle wait. That matches the sentence's existing scope. The gate wait and parent lock wait it leaves out were never in that sentence.
- **Cards 6 and 17 on `docs/overview.md`:** Card 6 edits the loom config paragraph and the loom.yaml reconcile paragraph. Card 17 edits the recipe lines, the landing paragraph and the module descriptions, and adds the reconcile sentence about `merge_fix_attempts` and the gate key. The two scopes don't overlap.
- **Card 12's new "What bounds it" paragraph:** Its mis-classing bound agrees with card 8's Intent ("`decision` … goes to the run's parent and never to the fixer"). The lets-through paragraph agrees with the card's Carry-over section: `decision` findings at any severity plus the review's BLOCKING findings go to `## Open risks`. It also agrees with the continue rule that applies only the dropped-key check.
- **Card 7:** The Edit list still names `TestConfigTemplate_DecodesAndIsSelfConsistent`, which the six-entry order check still edits. The "What it lets through" and bound paragraphs don't depend on the dropped sentence checks. `internal/lenses/config_test.go` has no existing sentence check for `goal-scope` or `board-fit` that would go stale.
- **Card 16:** Every new Edit glyph (`internal/gateslot#TestLoadConfig`, `internal/loomshed#roundGateConfig`, `internal/websterengine#verifyGateConfig`) is named in the fixture sentence. The three members of `mergein_test.go` are named only in the Intent, covered by the Rename pair, as `TestMergeIn_Fixer` already is. "webstercli's `verifyGateConfig`" is now distinct from "websterengine's `verifyGateConfig`".

Tool calls:
1. Read card 8's Intent, card 17's Intent and the waiting-note lines of `docs/overview.md`. My context held neither card 8 nor card 17's Intent, and it didn't show whether other overview lines name the waits.
