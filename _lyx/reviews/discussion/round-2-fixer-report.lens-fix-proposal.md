```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal check: I challenged each proposal the fixer carried out as written (F1, F2, F5, F6, F7) and built a case where it goes wrong.

Proposals fixed as proposed: F1, F2, F5, F6, F7. F3, F4 and F8 were fixed differently, so they fall outside this lens.

- **MEDIUM**, D8's second bullet, "left out of both BLOCKING checks … by the entry's class, the ledger's open-BLOCKING-entry check".
  F1's proposal exempts ledger entries by their `class`, but a ledger entry's class is optional.
  Case: the judge writes the BLOCKING `decision` finding's ledger entry with no `class`, which is legal.
  The ledger check still fires, every re-judge is refused the same way, and the segment halts at its budget.
  The amended criterion ("A BLOCKING `decision` finding alone … does not refuse the accept") then fails.
  Evidence: the `parseLedger` doc in `internal/shedadapters/bouncerfiles.go` says "An entry's optional class and severity, when present, must be burlerengine vocabulary".
  The `class` field in `ledgerEntryYAML` is unvalidated when absent.
  The record should match the entry to the round's `decision` findings by key or require the class.
- **MEDIUM**, D8's second bullet, "An `accept` over a re-judged CONTINUE or CIRCLING verdict passes the round as a circling accept does", against the amended criterion at former line 220.
  F2's re-judge-first proposal, applied to an `accept`, now lets the parent's answer to a `decision` question pass a BLOCKING `design` finding.
  Case: the review holds BLOCKING `design` finding X and `decision` finding Y, and the judge wrongly returns CONVERGED.
  The parent records `accept` to answer Y. The refusal re-judges, the new verdict is CONTINUE, and the accept passes the round, carrying X unfixed into `## Open risks`.
  Before the fix the record said such an accept "is not passed" and ran another round over X.
  D8 also says "the decision is the parent's answer to them", meaning the `decision` findings, not X.
  The brief does list X, but nothing tells the parent that an accept will pass X.
  The brief's way-forward note says only "the round is then re-judged before the recorded answer acts".
- **LOW**, D10's added gate sentence, "with the gate's findings kept and the stop report in the change description for whoever resumes it".
  F6's proposal halts through the exhausted `discussion` gate.
  The halt's kept findings then name an absent decision record, not the outdated entry.
  Nothing in the halt points the reader at the stop report in the change description.
  A resume with a fresh budget re-prompts the writer, which stops again, until someone happens to read the description.
  Evidence: the recipe header in `contracts/recipes/darn-recipe.yaml` says "an exhausted writer gate halts the run for the parent", and `shedrecipe.GateFindingsFile` holds only the gate's own findings.

F5 and F7: I found no case where the proposal goes wrong.

Tool calls:
- I grepped the `settledGeneration` doc in `internal/shedadapters/bouncer.go` to check whether a clear could leave F5's per-round snapshot file stale. My context lacked the clear path's file handling. It says a clear wipes the run directory, so there is no finding.
