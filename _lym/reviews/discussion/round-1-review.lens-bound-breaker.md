```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's claims and bounds.

**Claims attacked**
- D3, the six picks for weights 2 and 1 give A B A A B A: held.
- D3, two picks at once leave one entry and one increment: held, since both run under the file's lock.
- D3, a new or reweighted account's count is reset from the unchanged accounts' lowest count over weight: broken. A retired account divides 0 by 0 (F4).
- D4, a pinned worktree "can run elsewhere while its pin is out": broken for every pinned worktree that has no entry (F1).
- D4, the `~/.claude` account gets `CLAUDE_CONFIG_DIR` unset, found by comparing cleaned paths: broken by a symlinked `config_dir` (F8).
- D4 and D8, the scrub stops an operator's shell `CLAUDE_CONFIG_DIR` reaching a pane: held.
- D5, the forged-input bound "one listed account's figures in one window": broken (F3).
- D5, an opened session's status line "still records figures": broken when creel hooks are off (F5).
- D5, the merge's later-reset and higher-percentage rules: held.
- D6, the copy runs before the live-holder test: held for the interleavings I tried, since a live writer is always the newest copy.
- D6, sharing `plugins/` avoids the prefix-check 400: broken by D10's own optional settings link (F6).
- D7, two sessions hitting a limit at once make one switch: held.
- D7, the relaunch "the way shuttle's own respawn replaces a strand": broken, because no in-run strand respawn exists (F2).
- D7, the `set` bound "any session in the worktree": understated (F7).
- D1, a retired account is "never picked or switched to": partly broken, since entries already on it keep launching there (F9).
- D2, a wiped file reads as today: held.

**Findings**

**F1. MEDIUM, design. D4 and D7: a limit on a pinned account sends every other worktree on that pin to no account.**
- `blocked_until` is per account, but only the worktree that hit the limit gets an entry from D7's switch.
- D3 never picks a pinned worktree or the prime, so their sibling worktrees on the same pin have no entry.
- D4 therefore resolves each of them to "none". It exports nothing, and the session runs on the `env` value or `~/.claude`.
- That is either the blocked pinned account itself, when it is `~/.claude`, or an unlisted directory. It is never an eligible account.
- Counterexample: a hub pin P. One task worktree hits the limit. Every other worktree's next launch runs on `~/.claude` until `blocked_until`.
- Evidence: `_lym/discussion/decision-record.md:79`, `:101-102`, `:178`.

**F2. MEDIUM, design. D7 and the plan-writer notes: the relaunch rests on an existing mechanism that does not exist.**
- D7 says the relaunch works "the way shuttle's own respawn replaces a strand".
- Shuttle has no in-run strand respawn. A respawn is a caller archiving a run and starting a new one after `Attach`'s respawn-eligible verdict, and `Attach` "never relaunches or moves a strand".
- `retryAPIError` re-sends a line into the same pane and launches nothing.
- So the in-run relaunch, and its guarantee of no stopped outcome and one session-end record, is a new mechanism with no precedent. The record does not say how it keeps the completion-signal rule.
- Evidence: `internal/shuttleengine/attach.go:413` and `:499-501`; `internal/shuttleengine/apierrorretry.go:1-2` and `:41-47`; `_lym/discussion/decision-record.md:180-181` and `:304`.

**F3. MEDIUM, design. D5's forged-input bound is understated.**
- The verb takes any listed account's name as an argument, and the input carries both windows.
- So one shell can forge both windows of every listed account, one call per name.
- Every account then reads short until the forged resets, up to seven days, and D3 falls to step 3.
- A forged later reset also replaces real stored figures, by the merge rule.
- The bound says "one listed account's figures in one window".
- Evidence: `_lym/discussion/decision-record.md:127-132` and `:142`.

**F4. MEDIUM, design. D3's count reset divides by zero for a retired account.**
- A weight-0 account listed throughout is "listed then under an unchanged weight", so it enters "the lowest count over weight". That gives 0/0.
- In Go a NaN compares false, so the minimum is silently wrong.
- The rule does not exclude weight-0 accounts.
- Evidence: `_lym/discussion/decision-record.md:43` and `:90`.

**F5. LOW, design. D5: an opened session can have no settings file to carry the status line.**
- Claude's `OpenLaunch` writes `settings.json` and passes `--settings` only when the creel flag directory is non-empty.
- Without creel, an opened session has no settings file, so no status line and no figures.
- D5's claim that "its status line still records figures" then fails, and D5's "the settings the engine passes with `--settings`" names nothing.
- Evidence: `internal/shuttleengine/claudeengine/command.go:474-480`; `_lym/discussion/decision-record.md:126` and `:143`.

**F6. MEDIUM, design. D6 and D10: the precondition that keeps the prefix check from firing is optional in D10's own wiring.**
- D6 says the 400 fires when the tools or system prompt differ, and that sharing `plugins/` avoids it.
- `enabledPlugins` lives in each directory's `settings.json`, and MCP servers in each directory's `.claude.json`. D10 links the settings only "as wanted" and never links `.claude.json`.
- So two accounts with different enabled plugins or MCP servers load different tools, and by D6's own claim the resume fails.
- Evidence: `_lym/discussion/decision-record.md:164` and `:228-229`.

**F7. LOW, design. D7's `set` bound is understated.**
- The verb resolves its worktree from the cwd.
- Any process on the machine that can reach the hub can run it from any worktree, for example an agent that changes directory into a sibling worktree.
- The bound says "any session in the worktree".
- Evidence: `_lym/discussion/decision-record.md:186-188`.

**F8. LOW, design. D1 and D4: a symlinked directory defeats the `~/.claude` unset.**
- Directories are compared expanded and cleaned, with no symlink resolution.
- A `config_dir` that is a symlink to `~/.claude` is not seen as `~/.claude`, so D4 exports it.
- That is the `.claude.json` move D4 itself warns against.
- For the same reason, two entries can name one directory.
- Evidence: `_lym/discussion/decision-record.md:41` and `:105-106`.

**F9. LOW, design. D1: retiring an account does not stop its use.**
- D4 reads an entry regardless of weight.
- So worktrees whose entry already names a retired account keep launching on it until a limit switch or a `set`.
- D1's "never picked or switched to" leaves that path open, and the record does not say so.
- Evidence: `_lym/discussion/decision-record.md:43`, `:67` and `:101`.

**Tool calls**
1. `internal/shuttleengine/claudeengine/command.go:455-492` and the load function in `config.go`: my context lacked whether `OpenLaunch` always passes `--settings`, and how the claude module loads its config.
