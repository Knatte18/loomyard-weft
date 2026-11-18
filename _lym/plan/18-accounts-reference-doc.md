# Card 18 — How to wire a second account, and the manual check

**Create:**
- `docs/reference/claude-accounts.md`

**Intent:**
D10's operator half, which the landing notes point to (the Review criterion).
The page states, in order:
- what the account keys are and that they live only in the machine layer, set with `lym config claude --layer machine` (D1), with an example `accounts` list of two accounts, their weights and windows, and the two pins;
- how to wire a second account on a machine: log it in with `CLAUDE_CONFIG_DIR=<dir> claude` and `/login`; link `<dir>/plugins` to `~/.claude/plugins`, so lym's deploy, which syncs plugins into `~/.claude`, serves both; link `<dir>/settings.json` to `~/.claude/settings.json`, so both carry the same `enabledPlugins`; copy the MCP servers of `~/.claude.json` into `<dir>/.claude.json`, so both accounts load the same tools, which D6's copied transcripts need; link the user-level instruction file as wanted;
- that the operator owns logging in, preparing directories, and whether the use is within the accounts' and the organisation's terms;
- `lym claude account show` and `set`, the status line, and that an operator's shell `CLAUDE_CONFIG_DIR` no longer reaches lym panes, the `env` module being the way to set one for every pane (D4);
- the manual check, run by the operator on OSL-1046 after the task lands on neve and reaches OSL-1046: start two task runs, confirm their worktrees' entries with `lym claude account show` and that a transcript lands in each account's `projects/`, then switch one worktree with `lym claude account set` and resume a session there; it is the first run of a copied transcript resuming under the second account.
No test asserts the page's text: its wording is checked by the review reading it.

**Commit:** 18: docs: wiring a second Claude account and the manual check
