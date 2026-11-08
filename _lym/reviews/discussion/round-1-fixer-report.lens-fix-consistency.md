```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling decisions, criteria and cross-references they touch (fix-consistency lens).

**1. MEDIUM: a discarded package escapes D5's `--max` and time bounds**
- Changed text: D5 says "A skipped package does not count against `--max`." It also says a package whose HEAD or hashes changed under its tests is discarded "and reports the package as skipped".
- Issue: a package discarded this way has already had its coverage build and its tests run. Because a skip does not count against `--max`, a call can build and run more than `--max` packages.
- Evidence: two sentences of D5 no longer hold.
  - The bound line: "holds one slot for at most `--max` packages per call".
  - The new time bound: "per package it builds, one coverage build and its tests' count times `redundancy_test_timeout_sec`". It assumes the built packages are capped at `--max`.
- During a run of landings, the call keeps building and discarding with no cap.
- Possible fix: count only the dirty and locked skips (made before any build) outside `--max`. A discarded package would still count.

**2. LOW: D7's skip rule borrows D8's match, but D8's match is defined on HEAD**
- Changed text: D7 says "It is skipped when the worktree's existing report matches the current files by D8's match".
- Issue: D8 defines its match as "step 1 of D7 run on HEAD" against "the reviewed HEAD's tree". The gate step judges the uncommitted current files.
- Nothing says how D8's match applies to the working tree, so the reference is loose.
- A report that judged nothing because there is no parent branch records no hashes. Whether it matches, so the step is skipped, is also left unsaid.

**3. LOW: the D6 criterion contradicts itself when both conditions hold**
- Changed text: the D6 criterion says "It starts none when the last refresh only skipped packages".
- Issue: the sentence before it says a refresh starts when the prime HEAD differs from the record. If the last refresh only skipped packages and HEAD has also moved, the two sentences disagree.
- The skip-only rule needs "and the prime HEAD is unchanged".

**4. LOW: the support log's question ledger was not updated with the record**
- Changed text: the support log's round-1 entry, and the D5 and D7 changes.
- Issue: three ledger entries no longer match the record.
  - Q5 still answers which keys run the step without the reinterpretation that now carries it to hubs that tune those keys.
  - Q7 ("Limits on the refresh") lists only `test_map_batch` and `test_map_max_age_days`, but D5 now gives each test `redundancy_test_timeout_sec`.
  - Q6 still lists `redundancy_test_timeout_sec` as the step's limit only.

No further tool calls made.
