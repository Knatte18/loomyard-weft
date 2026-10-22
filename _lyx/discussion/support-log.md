# Support log — decision-add-notifies

## Interview

A darn run holds no interview: no operator answers, and every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — exploration

- Read the board entry, the verb in `internal/loomcli/decision.go` and its tests, creel's Go client and its sender-identity rules, `orchcli.NotifyPrime`, and shuttle's live-run reader `ReadAgentActivity`.
- Creel types a program's notice in full when the receiver is idle, and accepts a program claim only from an orch pane or a process in no pane.
- Shuttle's run records name each live run's strand; the tackler and the arbiter are such runs under their own roles.

## Rejected alternatives

- Notify only the run's status `current_producer`: a row name does not map to a strand name, and a MultiLLM row or a webster stage holds several live sessions.
- Filter live runs to those tied to the current shed step: a stopped step's runs are what the tackler handles, and untied runs exist; the role filter is simpler and covers the board's two exclusions.
- Carry the decision text in the notice: a receiver that trusts only its parent's messages would have to judge it, while the committed record is already trusted.
- Fail the verb when the notice fails: the decision is already committed, and a refusal would invite a duplicate entry on re-run.
- A `--no-notice` flag: no caller asks for it, and one idle-gated line is cheap.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, self-pick): Does a flag turn the notice off? No (D5).
- Q2 (resolved, self-pick): Who receives the notice? Every live LLM run of the target worktree but the tackler, the arbiter and the caller (D1).
- Q3 (resolved, self-pick): Does a failed notice fail the verb? No; it is reported on the success envelope (D4).
- Q4 (resolved, self-pick): Which identity signs the notice? The LOOM program, or the caller's strand when the caller is a named non-orch strand (D3).
- Q5 (resolved, board): The orch role's Fixing a run section drops the separate message step.
