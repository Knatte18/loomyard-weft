```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as written (F1, F3–F12; F2 was fixed differently).

1. **MEDIUM.** D5's new bound sentence: "A forged input can only lower one listed account's eligibility in one window, and only until that window's reset, at most seven days on."
   - **Summary:** under D5's own merge rule, a forged input can also raise an account's eligibility, so the bound F5 proposed is false.
   - **Evidence:** D5 says "an incoming window with a later `resets_at` than the stored one replaces it".
   - **Case:** the stored `five_hour` is 95% used and resets at now+2h. A forged call sends `used_percentage` 0 with `resets_at` at now+4h59m. That passes the new range check, because it is within five hours. It replaces the stored window, so an account at its limit reads as having headroom and gets picked.
   - The bound should cover both directions: one listed account's figures in one window can be set either way until the forged reset.

2. **MEDIUM.** The D2 relaunch bound ("until then a relaunch keeps the old directory, and the refusal of a gone directory above does not run for it") together with the D2 bullet "A record whose directory exists is kept".
   - **Summary:** F7's option 1 assumes a relaunch onto a gone directory only skips the refusal. But Claude Code started under a missing `CLAUDE_CONFIG_DIR` writes its state there, so the directory comes back.
   - **Evidence:** D4's facts say `.claude.json` and every `~/.claude` path move under `CLAUDE_CONFIG_DIR`.
   - **Case:** the operator deletes account B's directory, and reed relaunches a strand. Claude recreates the directory, empty and logged out, and the strand stops at the login prompt. The next fresh launch then finds a record whose directory exists, keeps it and never re-picks. D2's "picks again" for a gone directory, and the refusal's way forward "restore the directory, or remove the record", stop applying without notice.
   - The bound should state this, or D2 should judge a directory by its credentials, the same test D3 step 2 uses, rather than by its existence.

3. **MEDIUM.** The D5 rationale: "a renamed account, or a name reused for another directory, files its figures under the entry that names the directory now".
   - **Summary:** F10's resolve-at-launch keeps this promise only for sessions launched after the rename.
   - **Evidence:** D5 resolves the name "at launch" and bakes it into the settings' `statusLine` command. The verb then checks only that the name is in the current list (F5).
   - **Case:** account A (directory X) is renamed C, and a new entry A is added on directory Y. A session launched before the rename still passes "A". The verb accepts it, because A is listed, and files X's figures under the new A, which is the cross-filing F10 was meant to remove.
   - The same happens to a running session after a plain rename: the verb refuses the old name, so that session's figures are lost until it is launched again.
   - The rationale should state that bound, or the verb should resolve the account by the status line input's own directory.

4. **MEDIUM.** D2's fresh-launch search (F1): "first searches every known directory (D6) for transcripts of this worktree's sessions … records the one holding the most recently written transcript and makes no pick".
   - **Summary:** Claude Code keys transcripts by the worktree's path, not by the run. The search therefore finds a path's earlier runs and skips the weighted pick for any task worktree at a reused path.
   - **Evidence:** D2's rationale says "A per-worktree record dies with the worktree". The transcripts do not: D6 relies on them surviving after landing ("a landed run's sessions").
   - **Case:** a slug is re-run, or a task worktree is recreated at the same path. Its first fresh launch finds the earlier run's transcripts and records that account with no pick and no counter increment. The default directory holds pre-change transcripts for every path used before this change, so each such path stays on the default account for good. This is wider than the prime and beam worktrees F1 was meant to cover.
   - The search should be limited to transcripts that a resume of this worktree's sessions could need, or the bound should be stated.

I made no further tool calls.
