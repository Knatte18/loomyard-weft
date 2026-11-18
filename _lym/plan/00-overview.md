---
format: 5
approved: false
language: go
---

# Plan: Claude account per worktree, picked by weight and headroom

Spread Claude usage over the accounts logged in on one machine.
Each worktree runs its Claude sessions under one account, picked by weight and headroom when the worktree is made on the machine, pinnable per hub or for the prime and beams, and switched to another account when its own runs out unless a hub pin holds it.
The whole account policy is machine-local: the `claude` module's machine layer, a hub-wide account file and per-account usage records under the Bolt's `.lym`.
The cards build bottom-up: the shell statement and the config keys, the told worktree geometry, the stores and rules, the launch export and readers, the verbs and status line, then reed's hold ops, the relaunching record, the switch, the in-run relaunch, and the orch watcher.

## Card Index

1 — shell-unset-env — add `Shell.UnsetEnv`, the statement a launch on `~/.claude` uses to unset `CLAUDE_CONFIG_DIR`
2 — claude-account-config — the `accounts` list and `account_pins` as machine-layer keys of claude.yaml, validated and refused from tracked layers (D1)
3 — engine-worktree-geometry — tell launching engine sets the worktree's name, kind, account directory and names reader (D4's told geometry)
4 — claude-account-store — the account file, the usage records, the pick, headroom and resolution as pure functions (D2, D3, D5, D4)
5 — account-pick-at-making — pick at Worktree-Create and adopt through the new `AccountKeeper` capability (D3, D9)
6 — launch-exports-account — every launch resolves the account, exports or unsets `CLAUDE_CONFIG_DIR`, records it, and reed's scrub drops it (D4)
7 — known-config-dirs — readers of Claude's config directory read every known directory, and the reader builders get the hub's dirs (D8)
8 — resume-carries-transcript — a resume first copies the newest transcript into its account and records the copy (D6)
9 — claude-account-verbs — `lym claude account set|show` and the status line verb (D7's verb, D5's verb)
10 — status-line-settings — sessions on a listed account get the lym status line in their settings (D5)
11 — reed-hold-and-relaunch-ops — reed ends a strand's process while holding it and relaunches it under the same GUID (D7's reed ops)
12 — run-relaunching-record — a run record reads relaunching through one running predicate every reader takes (D7)
13 — account-switch-on-limit — record a limit hit's `blocked_until` and switch the worktree's entry (D5, D7)
14 — in-run-relaunch — shuttle relaunches a switched session inside its run through the stop verb's relaunch mode (D7)
15 — relaunch-bound-on-attach — a relaunch whose waiter died is ended or resumed once its bound passes (D7)
16 — refused-resume-starts-fresh — a copied resume whose first turn ends on an API error starts fresh in the run (D6)
17 — orch-switch-and-fresh — the orch watcher switches the orch's account and restarts it, fresh on a refused resume (D7, D6)
18 — accounts-reference-doc — how to wire a second account and the manual check (D10)

## Shared Decisions

- The told worktree is `shuttleengine.Worktree`, carried by `shuttleengines.NewForWorktree` and `claudeengine.NewForWorktree`.
  `New` and `NewFromConfig` keep their signatures and mean "told no worktree", which resolves only the hub pin, so read-only wiring and tests are untouched.
- The account directory is told as `Worktree.AccountDir`, which `fabricengine.ClaudeAccountsDir` builds from `lymdirs` constants (`PATTERN-cwd-resolution`, `PATTERN-lym-files`).
- The run directory's launched-account record is `account.json`; the copy fact is `RunState.TranscriptCopied`, in the same run directory's `run.json`, so the runner writes it without knowing an engine's file.
- The account file keeps D2's `worktrees` map and adds a `picked` map of entry times, which makes two picks at once for one worktree leave one entry and one increment (D3's criterion).
- The live-transcript readers (fork audit, outstanding forks, session-name drift, session usage) read the launched directory (D8) as the known directory whose session registry holds the session's live entry, since shuttle hands them only a session id and a workdir, not the run directory holding `account.json`; a session with no live entry reads under the worktree's resolution, then the other known directories (card 7).
- A new beam is made by its pawl run's Worktree-Create step before Beam-Up marks it, and `hubgeom`'s `pairIsBeam` already reads the board's beam flag for that window, so `EngineWorktree` reads it as a beam and no pick runs for it under a beams pin (D3).
- `PATTERN-shuttle-provider-seam`'s overview line stays as it is; card 14 fences it and changes the background's Shuttle stop section, so the overview does not grow.

## PATTERN changes

- `PATTERN-shuttle-provider-seam` — changes: the Shuttle stop section admits the stop verb's relaunch mode (D7): it marks the run record relaunching and keeps the strand; until the mark clears or its bound passes, every running-run reader reads it as live, a step stop stops it, and the orphan sweep keeps it.
