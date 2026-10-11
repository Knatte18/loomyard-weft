---
round: 1
focus:
  - "D1 vs D5 in decision-record.md: D1 says an empty `tmux_conf` leaves servers behaving exactly as today, while D5 adds unconditional `exit-unattached off` and `destroy-unattached off` pins on every server start. Settle whether the two agree, and whether the acceptance criteria cover the D5 pins."
  - "D2 and D3 in decision-record.md: path validation runs on every POSIX boot, sourcing only on the boot that starts the server. Settle whether the decisions state which boot is 'the one that starts the server' unambiguously enough for the plan writer, including two worktrees booting concurrently, and whether the acceptance criteria exercise that."
  - "Attack-surface question for D3, D4 and D5 in decision-record.md: for each guard or edge (arbitrary commands via `source-file`, non-fatal source errors, the open-ended 'plan writer adds any other option' clause in D5), settle whether the Decision states what it lets through and what bounds it."
  - "Relocation candidates: check whether the 'Auto-mode assumptions' bullet about the explored machine's missing `~/.tmux.conf`, and the Open risks entry about tmux's `~` and glob handling, carry a requirement the plan writer needs. Apply the completeness-before-leanness test before suggesting any move to support-log.md."
  - "Open risk about unknown sessions (Q11 in the ledger): settle whether handing the confirmation to the plan writer is a requirement that belongs in Decisions or Constraints, or correctly stays an Open risk."
  - "Consistency between decision-record.md and support-log.md: do the Rejected alternatives and the Question ledger match the Decisions as written (D1 through D7), with no ledger answer contradicting a Decision?"
---

Seed pass.
The pair is a single-feature design with seven Decisions.
Focus targets cross-decision consistency and bounds rather than missing enumeration; the Notes for the plan writer, missing exact signatures and call-site lists are not findings per the rubric and the writer's stencil.
