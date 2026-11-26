```yaml
lens: fix-affected-set
outcome: reported
```

Task: checking that the fix covers every member of each set-premised finding (the fix-affected-set lens).

**Unchecked findings:** none.

**Findings**

1. **MEDIUM**: `internal/shellguard/rows.go:428`, `steerSessionBusDeny`
   - The rewritten steer lost the space after the colon and now reads `are refused here:the user session bus they reach ...`. The text was `here: the` before the fix.
   - The notice is derived from the steer, and the golden was regenerated from this code, so both should carry the typo and no test catches it.
   - Evidence: `git grep -o` on the file returns `are refused here:the user session bus`.
   - Fixing it needs another golden regeneration.

2. **LOW**: `internal/shellguard/rows_integration_test.go`, the new `hub-tmux-server` rows (F2's set)
   - F2 named the single-quote opening "after -L or -S". Only `-L 'lym-…'` got a row.
   - `-S '/tmp/tmux-1000/lym-…'` still has none. The pattern now has a separate `(\\"|')?` branch for `S`, and the only quoted `-S` row uses a double quote.

**Other sets**
- **F1:** the cluster class `[2CDlNuv]` covers tmux's value-less flags apart from those that only print and exit. Both `L` and `S` branches use it.
- **F2, scopes:** both guards now have double-quoted-script rows.
- **F3:** the steer, doc comment, row comments, `doc.go` and golden were all updated. The only other `dbus-send` mentions are historical research docs.
- **F4:** the callers of the shared client read are `reed switch`, the `{strand}` button and the RC click, and the blast radius names all three.
- **F5:** the sampler stencil is the only stencil that tells an agent to name a tmux server.

**Outside this change set:** `tools/sandbox/SANDBOX-*-SUITE.md` tell agents to run `tmux -L <socket>` against a hub server, which the new row would refuse in a lym session. This comes from the original diff, not the fix.

**Tool calls**
1. `git grep` for `dbus-send`: my context only had the pre-edit search.
2. `git grep` for `tmux -L`, `tmux -S` and `AllowRawTmux`: the earlier search covered only the stencils and sampler paths.
3. `git grep` for callers of the client-read functions: my context did not list them.
4. `git grep` on the steer's spacing: call 1 showed the missing space in truncated output, and I confirmed it.
