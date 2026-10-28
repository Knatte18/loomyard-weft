## Interview

A darn run holds no interview.

## Rejected alternatives

- Using the operator's `~/.codex` directly: rejected, case 5 needs a controllable trust list and the operator's config must stay untouched.
- Symlinking `auth.json` into the spike home: rejected, a token refresh would then write through to the operator's file at an unpredictable moment.
- Matching the prompt glyph to detect turn end: rejected by the board entry; a hook writes a file instead.

## Review rounds

_No rounds yet._

## Question ledger

- Where does the spike's `CODEX_HOME` live? Self-pick: the gitignored scratch area of this worktree, resolved: yes.
- Does the entry stay current? Self-pick: yes, no landed work or peer touches Codex hooks, resolved.
