```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1, F2, F4, F5, F6, F7, F8, F9).

**Findings**

1. **MEDIUM**, `docs/overview.md`, codex paragraph (F2 fix). It says each plugin hook is "bounded by `codex.yaml`'s `plugin_hook_timeout`", but that is only the default.
   - `pluginSessionStartContexts` in `pluginhook.go` takes `hook.timeout` when the hook sets one and falls back to `req.HookTimeout` only when it is zero.
   - The test row `JSONGivesItsAdditionalContext` pins this: its hook's own 7s timeout wins over the request's 10s.
   - The sentence should say the setting bounds a hook that sets no timeout of its own.

2. **LOW**, `unreadScanSkippedDirectories` in `instructionfiles.go` (F5 fix). The skip matches by directory name at any depth, root entry included.
   - A checkout whose root directory is itself named `vendor` or `node_modules`: `WalkDir` gets `SkipDir` on the root and scans nothing, so no pointers are produced.
   - A session anchored inside a first-party `vendor/` directory: the walk never enters it, so CLAUDE.md files below the anchor go unnamed. The name `vendor` is common for non-dependency code.
   - A fix that skips only below the root and never into an ancestor of `workDir` avoids both. The test row (`src/app/node_modules/dep`) passes either way, so it does not catch this.

3. **LOW**, `TestPrepare_SubagentTerms` (F7 fix). The loop `append([]string{tt.wantEffort}, tt.skills...)` checks `"high"`, `"low"` and the skill names as bare substrings of a passage the test builds itself with `subagentPassage`, not of the launch.
   - `"low"` or `"high"` can match unrelated prose, so the effort assertion could pass without the passage naming the effort. The launch-line assertion on the effort setting still holds.
   - Asserting `` `reasoning_effort` to `<effort>` `` would be exact, but that is the wording F7 asked to stop pinning. This is a weakness inherent in the proposal.

**No finding**
- F1: both the skill listing's walk and the plugin install match use resolved paths now.
- F4: a marked override at `workDir` carries the CLAUDE.md, expanded, whether or not an AGENTS.md exists.
- F6: the self-link row gives `ELOOP`, which is not `ErrNotExist`, and the Linux and macOS tests do pass.
- F8: the keep claims match the code.
- F9: `renderEveryWorktree` still skips only prunable and staging worktrees and still lists prime, beams and pairs, so the restored clauses are true.

**Tool calls:** none. Everything came from my context.
