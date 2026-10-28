```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's changes to the decision record agree with the sibling text around them (fix-consistency lens).

**MEDIUM: the acceptance criterion for the escalate prefix still covers every recipe**
- Changed text: the new criterion "A pawl fixer `escalate` at Finalize or Parent-Align halts with `LandingStopPrefix` and hands back to the orch."
- Summary: the criterion it sits beside was left unchanged and still says every recipe uses the new prefix, so the two criteria contradict each other for a pawl run.
- Evidence: the unchanged criterion reads "A fixer `escalate` at Finalize and at Parent-Align halts with the escalate prefix; `landingshed.HandsBack` is false for it, the envelope is not marked `hand_back`, and `arbiterstop.Check` admits it." It has no "on a loom or darn run", unlike the Goal, the Scope bullet and D3's Bound, which the fix did qualify.

**LOW: D3's opening sentence is not qualified, though its siblings are**
- Changed text: the new D3 paragraph "A pawl run keeps its escalate a hand-back: … on false the escalate halts with `LandingStopPrefix` as today …".
- Summary: D3's first sentence still reads as a rule for every recipe, and the pawl exception only appears three paragraphs later.
- Evidence: D3's first sentence reads "A fixer `escalate` at Parent-Align or Finalize halts with a reason opening with a new exported prefix …". The Scope bullet and D3's Bound now both say "on a loom or darn run".

**LOW: the support log still carries the old pawl claim as a fix**
- Changed text: the support log's new Turn 4 entry for F2.
- Summary: Turn 3 still records the old pawl resolution as a fix, with no pointer to Turn 4.
- Evidence: Turn 3 reads "walkthrough 3 (LOW): pawl's arbiter has no decision record — fixed: D4 says it hands on." D4 no longer says that. Turn 4 records the change, so a reader can follow it, but Turn 3 is left reading as the current design.

Tool calls: none; my context already held everything the check needed.
