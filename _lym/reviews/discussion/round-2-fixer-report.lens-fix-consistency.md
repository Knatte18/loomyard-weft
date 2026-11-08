```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 changes agree with the sibling decisions, criteria and cross-references around them (fix-consistency lens).

Findings:

1. LOW: support-log.md, Turn 6 bullet `bound-breaker 6 (admission by caller): fixed, D6 admits by shape.`
   The bullet is now out of date. D6's admission is no longer a shape check alone. It also requires the entry's run status before the write to name the `Raddle-File` producer, which is a row limit added for F5.
   Evidence: D6 now reads "The run lock admits the change only when the entry before the write holds a run status whose producer is `Raddle-File`, and the entry after the write equals the arras project…". D6's Bound now reads "The shape check and the `Raddle-File` producer are the whole gate".

2. LOW: decision-record.md, Acceptance criteria, the new line `D4: with the split perch's auto-accept key off, a spent-budget round whose open findings carry no design-wrong tag is accepted automatically…`
   This D4 criterion sits between the D2 criteria and `Review: D3`. Every other criterion runs in decision order, and the existing D4 criteria start further down at `D4: an arras run records perch mode…`.
   Evidence: the criterion order is D2, D2, D2, the new D4, then Review: D3, then D4, D4, D4.

3. LOW: decision-record.md, D6, the sentence added for F5 that ends "Raddle-File's retry then finds the entry filed."
   The sentence clashes with D6's retry text. That text says a retry that finds the converted entry filed writes only the mark, clears the status and writes the store. A conversion made by another writer would not have written the parts' record files, which raddle writes for every part. That gap belongs to the fix-claims lens, but the two D6 sentences do not agree on what a "filed" entry holds.
   Evidence: D6 says "the lock reads them from the write's result" and "Raddle-File's retry then finds the entry filed". The same D6 says a retry "writes the mark, then clears any run status on the entry, and then writes the store". `fileRaddleEntry` "writes the entry's decision record file" (internal/boardengine/raddlefile.go).

The other changes agree with the text around them. D7's goto way forward now matches D2's and D4's fresh-entry wording. The reason on D1's line 43 survives the Constraints cut. Removing the D9 Open risk matches the removed ledger line.

Further tool calls: none.
