---
round: 1
focus:
  - "internal/boardcli/runsettings.go `refuseBeamOrchOutsideBeam`: settle whether `ParentBeamOf` returns an empty string for a beam whose parent is the prime, and for a slug with no beam entry (the `ly:other:orch` case), so the named way forward is a valid creel target and the error path does not mask the original refusal."
  - "internal/boardcli/runsettings.go `refuseRunSettingsCaller`: settle whether every branch where a beam's orch is refused outside its beam now returns the new refusal, and which refusals (another role, a non-orch strand) keep the old text; check the decision record's Wanted line that the refusal names the way forward."
  - "contracts/orchrole/orch-role.md new paragraph: check it against the amended board entry's Decision (parent beam not prime, wish not order, parent decides placement, pass upward) and that the creel command and strand-name form match what `lym creel post` and `agentname` accept; check it sits where the shortened role puts messaging or working rules."
  - "internal/boardcli/verbs_test.go `stepRunSettings`: settle whether the new `wayForward` assertions and the nested-beam row duplicate coverage or pin the parent-orch target, and whether the added `inner` entry disturbs the other rows."
  - "internal/boardcli/doc.go and the `refuseRunSettingsCaller` doc: check the added sentences are accurate against the code and follow the repository's comment conventions in its constraints document."
  - "Scope: settle whether the diff changes any refusal or board-write boundary beyond the way-forward text, since the board entry says no flag lets a beam orch write outside its beam."
---

The decision record holds only the board entry under `## Goal`; the other sections are empty and are no measuring stick.
