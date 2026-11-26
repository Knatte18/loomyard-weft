# Study whether the hardener finds and fixes real faults

Adds `docs/research/hardener-works-study.md`, a study of the two fabric hardener campaigns from their records and commits,
and lists it in `docs/research/README.md`.
No code changes.

## What the study finds

- **Findings and fixes hold up.**
  No numbered finding is a false positive, and each fix cures the scenario its finding shows, with a regression test that by reading fails without it.
  Several severity ratings are high, and parts of some fixes have no test.
- **Several fixes brought new faults.**
  The most notable still open: a refused credential during a one-repository seed keeps the hub and says origin may still take the push.
  The report lists the other faults read in the code, none of them driven.
- **Convergence is the weak part.**
  The second campaign converged with a MEDIUM fault recorded only in prose, because an incremental round could not raise it.
  `8cf7f4775` does not close this: it covers undriven and WARN focus entries, not FAIL ones or faults seen outside them.
- **One reachable miss:** the `remove` pending record that names a command which cannot finish.
- **Cost** is tabled per round.
  The records give no evidence for how the current `sonnet[high]` default performs.

The report ends with a verdict and a list of changes to the hardener recipe and its stencils, each a candidate note.
No note is filed by this change.

## Design decisions

- No reproduction ran, as the decision record's added entry "Reproductions dropped" says:
  the report judges c1 F3, c2 F2 and c2 G3 from records and commits and states why.
- Every judgement of a test is by reading the test and the fix; no test or sabotage run was made.
- The report departs from the board entry on one point of fact:
  the round files show that a further round ran after the second campaign's round 3,
  and that it drove cases the entry and `docs/research/single-repo-undriven-cases.md` call undriven.

## Merge danger

**Door:** two-way
**Blast radius:** docs
