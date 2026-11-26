```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I'm looking for content to move out of the decision record or cut from it, then noting anything else I saw.

Findings:

1. LOW — `## Goal`, second paragraph ("Today most of these already survive… took them for gone"). The paragraph is exploration narrative: what survives today and why the analysis missed it. The plan writer needs only the gap list.
   - Completeness test: the list of what is lost (PR findings, ask record and reports, landing fixer reports) is a requirement. It is already restated in D1 to D3's rationale, so the narrative can move to the support log without losing anything.
   - Fix: cut the Goal down to the outcome plus the four deliverables.

2. LOW — D5's rationale, "the 1200-odd older ones included". This is a perishable tally: the tag count changes every landing. Fix: write "every archived run, older ones included".

3. LOW — `## Scope` Out, "Recovering records of runs whose artifacts were already lost". It excludes something no decision touches, so it is padding. Fix: cut it.

4. LOW — `## Auto-mode assumptions`. Each bullet restates D3, D4 or D6 nearly word for word, and the question ledger already records them as auto-picks. The section is required, so keep it but shorten each bullet to a pointer, for example "D4: the index row is per run, not a shared file".

5. LOW — `## Notes for the plan writer`. Both hints repeat facts already in D4 (`shedrun.LandedSizeRecord`) and D5 (`reportengine.Open`'s landings join). Fix: cut the section, or keep only hints that are not already in a decision.

6. MEDIUM — D2, "Each change to the parent-review ask record (`burlerengine.RecordParentAsk`, answer, expiry, clear, settle)". It lists every current writer of shared state, which is the enumerated-consumer-list form the writer was told not to gather. A new writer would silently skip the copy.
   - Fix: say "every write to the ask record copies it", and leave call sites to Plan.
   - Better still, state it as one mechanism: the copy happens inside the record's single update path.

7. LOW — D1, "The existing `.lym/loom/` copy, the friction note and the pull-request record stay as they are". This line about non-changes is fine as a bound. The next sentence, about teardown catching a late copy, repeats `## Constraints` bullet 2. Fix: cut that sentence from D1.

8. LOW — the `## Constraints` bullets cite PATTERN entries that the prompt already makes binding. That is acceptable, but the bullet on `structuralCommittedDirs` is supporting evidence, not a constraint. Fix: move it to the support log or `relevant.md`.

Other observations, not leanness:

- D5's acceptance criterion for `--slug` without `--extract` ("prints that slug's runs… newest first") has no bullet. Every D5 bullet tests the list or extract output, so the per-slug print is untested.

No further tool calls made.
