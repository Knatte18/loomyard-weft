# Decision for claude-accounts-per-worktree: pick the account with the most quota left

`decision-account-pick-most-left` · Note · enhancement, claude, config

## Decision (operator, 2026-11-18)

The pick at worktree making (D3 of the stored decision record) takes the account with the most quota left, not the next account by a weighted counter.

- Among the eligible accounts (listed, positive weight, logged in, with headroom in every window they list), take the one whose tightest window has the most left.
- The weighted counter stays only as the tie-break, and for accounts with no headroom reading yet.
- Pins are unchanged: a hub pin, then a beams pin, win over the pick, which covers the operator's cases of a repository always on account A, always on account B, and alternating repositories whose prime and beams stay on A.

Discussion rewrites D3 and anything that reads it (D2's counter, D5's headroom) to fit.
