---
round: 2
focus:
  - "In decision-record.md, D4 lets a per-worktree module's untracked local layer set any key, and states a bound that a local override can never change a key running a shell command or a hub-level fact shared across pairs. Settle whether that bound holds for per-worktree keys that name an executable or config sourced into the shared tmux server (reed.shell, reed.tmux, reed.tmux_conf, shuttle.claude), the standing agent guards (shuttle.claude_deny_*) and shuttle.run_dir, which must stay worktree-local; and whether the record states what bounds an override of them, given that an untracked layer is invisible at landing."
  - "In decision-record.md, D2 says a seed-only module's upper layers are only pruned, and the reconcile acceptance criterion prunes retired keys in every layer. configreg.Module.SeedOnly modules (models, burler) have operator-owned open-ended keys such as model aliases, burler lenses and fans, and configsync never rewrites a present seed-only file. Settle whether the record states a seed-only rule that keeps such keys in upper layers, and whether D5's unknown-key refusal and D8's key-path validation agree with it."
  - "In decision-record.md, D6 strips from a worktree, once, every key whose value equals what the layers below resolve to. Settle whether D6 states that a deliberate pin equal to the seeded default becomes inherited, so a later default-layer change alters that worktree, and what bounds or reports it."
  - "Check that any fix to the per-worktree local-layer bound or the seed-only rule leaves D5, D8, the acceptance criteria and the risks consistent with the new decision, and that no requirement the plan writer needs is moved into support-log.md only."
---

Focus targets the open gating design questions.
D9's removal window is LOW and not promoted.
