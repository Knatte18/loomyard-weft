# Support log — runs-never-write-board

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- A guard in each boardcli verb that inspects the payload's kind: rejected because it repeats per verb what the store decides, and misses the net effect of a merge or batch.
- Refusing every board write from a task session: rejected because a run may file notes, and the landing and run_settings writes stay allowed.
- Refusing an orch thread in a task worktree too, as the run_settings guard does: rejected because that orch is the operator's channel and would lose the ability to create tasks the operator approved.
- Detecting a run by the worktree being a task worktree: rejected because an orch thread and the operator also work in task worktrees.
- A shell guard row denying `lyx board upsert` to agents: rejected because agents may still file notes through the same verb.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Where does the refusal live? Self-pick: the board's write critical section, judged on the net change (D1).
- Q2: How is a run's session told apart? Self-pick: the strand name carrying a task slug, orch role exempt (D2).
- Q3: Which stencils change? Self-pick: Discussion, Plan, Darn writers, the discussion and plan rubrics, and the orch role (D4, D5).
