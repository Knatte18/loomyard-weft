---
verdict: CONVERGED
rationale: "Round 2 carries one LOW design finding (F1) on a key first raised this round, no BLOCKING and no gating finding at MEDIUM or worse; round 1's nine keys are each settled by the round's focus verdicts. No focus departures to rule on."
---

Round 2 raised one finding, F1 (LOW, design): `FollowTemplate` parses the template base even when the hub file is absent and `haveBase` is false, so a corrupt base fails a reconcile that would rewrite it anyway.
It maps to the new key `corrupt-base-parsed-when-ignored`, not to round 1's `corrupt-template-base-wedges-reconcile`, since round 2's own verdict on focus 1 settles that wedge: the error names the base file, and deleting it recovers the next reconcile.
No relabel was made.

The decision rule needs a BLOCKING finding, or a gating-class finding at MEDIUM or worse on a key open in an earlier round, to continue.
F1 is neither, so the loop has converged, and Go carries F1 forward as an open risk.

The round's focus questions settled the other round 1 keys: base commit paths, envelope keys, mail lines, sequence rows, base read failure and restore, long flow mappings, test doc comments and stale "never fills" wording.
