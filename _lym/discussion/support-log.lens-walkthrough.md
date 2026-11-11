```yaml
lens: walkthrough
outcome: reported
```

## Findings

1. **MEDIUM — D5: status line verb can fail to find the account, silently.** Case: account `config_dir: ~/claude-b/`; launch exports `CLAUDE_CONFIG_DIR=/home/u/claude-b`. The verb matches its env value against configured values; a trailing slash or a symlinked path means no account matches, so it writes nothing. D5 then reads an account with no record as full headroom, so the pick keeps choosing an account that is at its limit — purpose missed. Prepare (D4) already knows the account when it renders the session settings; rendering the account name into the status line command's arguments (or at minimum comparing cleaned absolute paths) closes this.

2. **MEDIUM — D3/D9: credentials gate on macOS.** Case: an account on macOS. The research found the login there lives in the Keychain keyed to the directory, and no `.credentials.json` exists. Step 2 finds no account eligible; step 4 fails the launch for every session. D3 doesn't scope itself to Linux; state the bound, or treat a missing file as eligible on platforms whose credentials live elsewhere.

3. **LOW — D3: new-account seeding at the highest ratio delays its first pick.** Case: A,B weight 1, counts 10 and 4; C added weight 1, seeded at 1×max(10,4)=10. B is picked 6 times before C gets its first pick; A and C then alternate from 10. Not a catch-up burst, but C gets less than its share for a while. Seeding at the lowest count-over-weight among listed accounts (rather than highest) gives the share from the next pick.

4. **LOW — D6: sessions under a prior shared config dir drop out of reports.** Case: before the feature, the operator ran every session under `CLAUDE_CONFIG_DIR` set in the env module. `lym report` reads every account dir plus `~/.claude`; if that env-module dir isn't also listed as an account, its sessions vanish from the report. D6 should add the env module's `CLAUDE_CONFIG_DIR` to the set it reads.

5. **LOW — D6: `tools/tokencount` locating accounts.** It sees the accounts only if it loads the hub's machine layer, which needs Board and Anchor in its config dirs; run outside a hub it reads the default dir alone. The record doesn't say how the tool finds the machine layer.

## Walkthrough list

- D1 (tracked `claude.yaml` holding `accounts`): purpose met — `configengine.Provenance` exists and names the layer each unit came from, so the refusal is buildable without the module reading layers itself.
- D2 (second reviewer session in a task worktree): met — reads the record, no pick.
- D3 (weights 2,1; six picks): met — A,B,A,A,B,A = 4/2, no two B in a row. Findings 2–3 apply.
- D4 (non-default dir account; default account with env-module value set): met — export in Cmd after env-module exports; ResumeCmd carries it too.
- D5 (session on an account with both windows): met only when the path match succeeds (finding 1). Prepare passes the settings via `--settings`, so the statusLine entry fits there.
- D6 (transcript lookup for a worktree on a fake dir): met for per-worktree readers; findings 4–5 for reports.
- D7 (ly:orch resumed after restart in prime): met — sticky record keeps the dir.
- D8 (session waiting on a usage limit then resumed): met — record unchanged, export same.
- D9 (manual check): met on Linux; finding 2 for macOS.

Tool calls:
- One grep of `internal/configengine` for layer provenance support — to test whether D1's tracked-layer refusal is buildable; my context had only the subagent summary of configengine.
- Same grep also covered `statusLine`/`--settings` in claudeengine — to check where D5's settings land.
