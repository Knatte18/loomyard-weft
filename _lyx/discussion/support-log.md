# Support log — webster-orch-need-to-parent

## Interview

A darn run holds no interview: the darn writer read the board entry, the issue's incident records (the gate-runs-once drive reports, halt notes, refusal notes and webster state commits) and the webster, shed and loom code, and made every call itself.

## Rejected alternatives

- A distinct shed halt type or anomaly kind for "waiting on the parent": the board's design choice rules it out.
- Keeping `FAILED` and teaching recover-batch to tell an orch need from a broken card: the report carries no reason, and a cold recovery spawn is the cost being avoided.
- Recording a waiting batch as terminal: every terminal non-done record reads as "needs recovery" to begin-batch, record-batch and the retry rule.
- Resuming a recovery batch through a fork: the fork re-begin drops the failed batch's suspect paths and failure digest.
- Refunding a recovery count when its strand reports waiting: the spawn that resumes the wait goes uncounted instead, which needs no record of whether the earlier spawn counted.
- Changing `RecoveryRetry`: the incident's two verdicts follow the rule for two different strands, with a batch-start reset between them.
- A shed rule that skips the producer call on a blocked row: a re-step of a blocked row is how a human resumes after fixing the cause, so the producer must run.
- Changing the shared "reports status: FAILED and the orch runs it" clause: other roles read it; the implementer body maps it for a webster strand instead.

## Review rounds

_No rounds yet._

## Question ledger

- Q: What does the fork report when it needs the orch? Self-pick: `status: NEEDS_PARENT` with a `need` line.
- Q: Is a waiting batch terminal? Self-pick: no; `Status` `waiting`, `Terminal` false.
- Q: How is the orch told? Self-pick: Master's own message to its parent, then a held turn end, whose hold notice also reaches the parent.
- Q: What makes the two recover-batch refusals consistent? Self-pick: the envelope names the strand's session; the rule stays.
- Q: Where does a repeated halt stop growing history? Self-pick: Run entry refuses the dead end with a fixed text, and shed folds a repeated blocking Stuck.
