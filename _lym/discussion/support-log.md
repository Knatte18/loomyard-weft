# Support log — burler-guard-readonly-git

## Interview

A darn run holds no interview; the writer read the board entry, issue #898's text in it, and the guard code in `internal/burlerengine/cluster.go`.

## Rejected alternatives

- Downgrading a guard hit to a retried attempt: rejected, since a true hit may have mutated the repository and a retry would build on that state.
- Ignoring every word starting with a digit and `>` only: rejected, since `>out`, `<in` and a target separated by a space would still leak into the operands.
- Adding an allow-list of read-only subcommands: rejected, since the guard already judges by a mutating-subcommand list and per-segment splitting; the allow-list would duplicate it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Is the board entry overtaken by commit 910ec75a? Self-pick: partly; the branch listing part is landed (D4), the redirection false positive and the regression test remain (D1, D2).
- Q2: Should a guard hit downgrade to a retried attempt? Self-pick: no (D3).
