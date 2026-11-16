```yaml
lens: interactions
outcome: reported
```

Interactions lens: checking pairs of decisions in the record that act on the same state, plus the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1×D5: notices and passages are keyed by channel; Claude uses its appended prompt and Codex uses `developer_instructions`. Held.
- D1×D7: broken. Finding 1.
- D1×D8: broken. Finding 3.
- D1×D9: broken. Finding 5.
- D1×D10: broken. Finding 3.
- D1×D4: the prompt-engine item has a neutral key and an engine value. Held.
- D2×D3: broken. Finding 2.
- D2×D5: "your developer instructions" matches where D5 puts the items. Held.
- D2×D6: the Codex block leaves `model` unset, sets `reasoning_effort` only on `fork_turns: "none"`, and inherits it on `"all"`. Held.
- D2×D6, thread cap: the six-fork lens review fits under 16. Held.
- D3×D4: the producer takes the directive's engine and the prompt engine from one model choice. Held.
- D4×D11: a hand-written prompt still passes, and only the board rule forbids it in benches. Held, as the record bounds it.
- D5×D8: the listing and plugin contexts go into `developer_instructions`, and the long-argv risk is recorded. Held.
- D5 order: broken. Finding 6.
- D6×D5: the subagent passage sits in developer instructions, and children inherit the config. Held.
- D7×D9: a larger `AGENTS.override.md` is covered because the cap is derived at Prepare. Held.
- D8×shared Codex home: broken. Finding 4.
- D9×D8: broken. Finding 7.
- AC1×PATTERN-test-speed: broken. Finding 8.
- PATTERN changes×D3: broken. Finding 2.

**Findings**

1. **MEDIUM — D1 vs D7, a contradiction on targets one engine reaches.**
   - D1 says there is "no exemption table: a content item without a Codex counterpart is a bug".
   - D7 keeps a repository entry's target "the repository's own choice" and keeps `claude-file` for Claude-only text.
   - A hub entry on `claude-file` or `claude-prompt` puts an item in the Claude manifest that Codex lacks. On that worktree, D10's verb would exit non-zero and D1 would call it a bug.
   - D1 should say how such items are keyed or left out of the comparison. One way is to compare only shared-target items and treat `claude-*`/`codex-*` targets as chosen per engine.

2. **MEDIUM — D3 vs D2 and the edit-rule guard: the shared edit directive's fate is unstated.**
   - D3 says the shared text "becomes the Claude copy" but does not say whether `contracts/stencils/edit/edit-directive.md` is removed.
   - If it stays, D2's scan fails on it, because it names Read, Edit, Write and `replace_all` outside `engine/claude/`.
   - `TestStencils_NoEditRuleRestatement` (`contracts/stencils/parentdirective_test.go`) exempts only stencils named `edit-directive`.
   - `editdirective.Directive` reads the `directiveStencil = "edit-directive"` constant.
   - The PATTERN-stencil-ownership background says "The `edit-directive` stencil is the only statement of the edit directive's rules". With one copy per engine that becomes two statements, and `## PATTERN changes` changes only the overview line.
   - D3 should name the file's removal and the guard's new exemption, and list the background change.

3. **LOW — D1 vs D10/D8: "Prepare builds the launch from the same manifest function" conflicts with the verb.**
   - D8 has Prepare run plugin hooks so it can deliver their output.
   - D10's verb "runs no plugin hook" yet uses the same function.
   - The manifest therefore has to key hook items by identity, not output. When a hook fails or times out at run time, the item is marked omitted, but the verb's verdict and the parity test still report parity.
   - The record should state that the manifest names identity and that delivery is a separate step.

4. **MEDIUM — D8: the shared Codex home's `AGENTS.md` versus content per worktree.**
   - D8 has `claudeengine` resolve the setup "a Claude session in a worktree gets" and write it to the single `AGENTS.md` in the shared lym Codex home.
   - The lock serializes the writes, but a session launched earlier reads whatever the last Prepare wrote.
   - If the global file differs per worktree, launches undo each other's copy. Board draft `claude-account-pick-simple` (two Claude accounts per worktree) would make the Claude config directory, and with it the global `CLAUDE.md`, differ per worktree.
   - D8 should state that the content is per user and identical across launches. Otherwise it should move the file to the per-session `developer_instructions`.

5. **LOW — D1 vs D9 and D8 bounds: carve-outs that hide gaps.**
   - CLAUDE.md files above the repository root, which D9 leaves uncarried, and unexpanded `@` imports (D8) are content Claude gets and Codex does not.
   - The parity check either flags them, which contradicts "no exemption table", or never lists them, so the manifest hides the gap.
   - The record should say which.

6. **LOW — D5: the "order Claude's appended system prompt holds" is undefined for two items.**
   - Claude delivers the skill listing and the plugin session-start contexts natively, not in the appended prompt.
   - The appended prompt also puts the subagent passage before the standing passages, while D5 lists the passages first.
   - AC "in Claude's order" therefore has no single reference order to test against.

7. **LOW — D9 vs D8: the instruction-file cap may not cover the global file.**
   - D9 derives `project_doc_max_bytes` from the files on the path from the repository root to the working directory.
   - It excludes D8's global `AGENTS.md`.
   - Whether Codex counts the global file against the cap is not established, so the cap may fall short.

8. **LOW — AC1 vs PATTERN-test-speed: the tier0 parity test must not run D8's hooks.**
   - AC1 says "removing any one Codex delivery fails the test", which includes plugin contexts.
   - Running hooks is a spawn, which tier0 bars, and Codex Prepare spawns the sandbox probe.
   - The constraint holds only if the parity test calls the manifest function and a delivery path with a faked hook runner. The record should say so, as it already does for the separate manifest function.

Tool calls:
- I grepped the edit-directive references in tests and in `internal/editdirective/editdirective.go`. My context lacked how the edit-rule guard and `Directive` name the shared stencil, which finding 2 depends on.
