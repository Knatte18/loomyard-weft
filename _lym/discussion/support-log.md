# Support log — fork-audit-branch-list

## Interview

A darn run holds no interview; the writer read the board entry and the fork audit in `internal/burlerengine/cluster.go` and decided alone.

## Rejected alternatives

- Treat the board entry's option list, `-a` and `-r` among them, as list mode whatever the operands: rejected because git creates a branch for `git branch -a foo`, so the audit would pass a writing command.
- A deny-list of writing options with everything else read-only: rejected because an option the list misses would let a writing command pass silently; the allow-list fails loud instead.
- Treat any short-option cluster as mutating, as `git tag` does: rejected because `git branch -avv` is a common listing form a fork would hit.

## Review rounds

_No rounds yet._

## Question ledger

- Do `-a`, `-r`, `--format` and `--sort` alone put `git branch` in list mode? Resolved by self-pick: no, they read only without an operand, following git's own mode selection.
- Should an unknown option count as mutating? Resolved by self-pick: yes, the allow-list fails loud.
