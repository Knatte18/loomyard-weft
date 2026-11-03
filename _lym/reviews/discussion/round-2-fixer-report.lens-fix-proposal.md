```yaml
lens: fix-proposal
outcome: reported
```

The fork-proposal lens: I challenged the reviewer's proposals that were carried out as written (F1–F11).

Proposals challenged: F1, F2, F3, F4, F5, F7, F8, F9, F10, F11 (F6 is a deletion with nothing to challenge).

- **MEDIUM** — D3's new clause "so the assistant never outlasts the chair's wait (D1)" and D1's Bound sentence "its start and one await cap fall in the margin between the two".
  The F8 proposal only requires the assist timeout to be "below" `review_timeout_min`, and that does not keep the chair's wait inside the chair's timeout.
  Concrete case: `review_timeout_min: 240` and `assist_reviewer_timeout_min: 239`. D1's own bound puts the chair's wait at the assistant's start, with its start retries, plus its timeout plus one await cap. Start retries alone are five seconds each up to `start_retries`, plus launch time, and the await cap is the role's keep-warm cap. That adds up to more than the one-minute margin, so the chair times out inside its wait.
  The guarantee only holds at the defaults. Validation would have to take the start allowance and the await cap into account, or the claim should say it holds only when the margin covers them.

- **LOW** — D6's new writable-root sentence, after the F1 proposal dropped the `/tmp` design and its macOS Open risk.
  The record now depends on `-c 'sandbox_workspace_write.writable_roots=[…]'`, which is marked "to be proven in the build". No Open risk says what happens if the setting does not take.
  The Constraints already record that another `-c` session flag, `projects.<path>.trust_level`, is ignored. If `writable_roots` is ignored too, every assistant fails to write its report: the gate fails, or the run times out. The fix removed the old fallback without stating this new failure mode.

- **LOW** — D5's new `.codex/rules/` refusal, which also covers "its main checkout root".
  Codex reads project config layers from the session's own tree. Trust is keyed on the main checkout, but the main checkout's `.codex/` folder is a different directory from the linked worktree's.
  So a `.codex/rules/` that exists only in the main checkout, untracked or on another branch, is never loaded by the worktree session. The refusal still stops the assistant over it. This case comes from the F4 proposal ("a worktree or main checkout holding .codex/rules/").

- **LOW** — D12's new rule that `codex-file` carries every repository-layer entry aimed at `claude-file`, while the shipped entries stay out "because they direct Claude's Read, Edit and Write tools".
  The F5 proposal sorts entries by layer, but the reason it gives is about content. A repository entry aimed at `claude-file` can name Claude tools just as easily, and it is now rendered to Codex without any check. The stated reason does not support the rule as written.

Tool calls:
- One Bash call resolved where `.lym` and `_lym` point (`codex-engine-weft`, a weft per worktree, so the F2 bound holds) and grepped for `review_timeout_min` in the tapestry and loom templates (only `loom.yaml` has it, so the D3 comparison stays in one module). My context had neither the link targets nor the template contents.
