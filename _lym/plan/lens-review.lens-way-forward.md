```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the plan in `_lym/plan` (cards 1–10).

Per refusal, the next step it names and whether that step can succeed from the state the refusal leaves:

- Card 2, `Show(agents-prompt)` refusal: names `lym heddle show <target>` with `claude-prompt` or `codex-prompt`. Progresses.
- Card 3, render warning from `NotifyWorktreeMadeAt`: names `lym heddle render` in that worktree, through `seamWarning`. Progresses.
- Card 4, failed write of the import record: no next step named. See finding 5.
- Card 4, unreadable import record: no next step named. See finding 5.
- Card 5, refusal over the 128 KiB argument cap: names the largest item and its way forward. A standing-passage item does not progress (finding 2).
- Card 6, `max_subagent_threads` below 1: `loadConfigOrRefuse` adds `lym config codex`. Progresses.
- Card 6, an effort codex.yaml does not list: a warning that names `lym config codex`, and the launch goes on. Progresses.
- Card 8, a hook that fails or times out: logged at Warn and left out, with no next step named. See finding 6.
- Card 9, `plugin_hook_timeout` invalid: `loadConfigOrRefuse` adds the way forward. Progresses.
- Card 9, user-setup source error: no next step named. See finding 3.
- Card 9 and card 8, cap refusal on a plugin context or the skill listing: the named step cannot succeed (finding 1).
- Card 10, walk or stat error: no next step named. See finding 4.

Findings:

1. MEDIUM — D5 and D8, card 8 and card 9: the way forward "turn the plugin off in claude.yaml's `session_scope`" cannot succeed.
   - `ScopeConfig` has only `Enabled` and `SkillsOff` (`internal/shuttleengine/claudeengine#ScopeConfig`).
   - `resolveSessionScope` turns a plugin on exactly when the session preloads one of its skills. The doc comment on `SkillsOff` says a plugin skill named there stays listed.
   - So no `session_scope` edit turns off a plugin whose skill the role preloads. The step that works is to drop that skill from the role's skills in agents.yaml, or from the spec.
   - With the scope disabled, card 7 counts every installed plugin as on. There the step is to enable `session_scope` (or uninstall the plugin).
   - The skill listing's way forward, "preload fewer plugins' skills in the session scope" (card 9), has the same flaw: skills are preloaded through agents.yaml and the spec, not through the scope.
   - Fix: word both ways forward around agents.yaml role skills, the spec's skills and `session_scope.enabled`.

2. LOW — D5, card 5: the refusal names a standing passage only as `heddle passage <n>`, with the step "shorten that heddle entry with `lym heddle set`".
   - `PassageReader` returns bare texts (`internal/shuttleengine/passages.go`), so the operator is never told which slug to shorten.
   - A shipped entry is changed only through `--layer lym` in a hub whose heddle.yaml sets `lym_override` (`refuseShipped`, `SetOverride`). A plain `lym heddle set` on a shipped slug is refused.
   - The step can succeed only after the operator works out which entry it is. It should say to find it with `lym heddle show <engine>-prompt`, or to override a shipped one with `--layer lym`.

3. MEDIUM — D8, card 9: a user-setup source error refuses every Codex launch with only the `codexengine:` prefix and no way forward.
   - Card 7 returns a read error of an instruction file. It does not say whether an unreadable or malformed `installed_plugins.json`, or a malformed SKILL.md frontmatter, is an error.
   - Claude's `sessionScopeFor` logs a bad plugins file and goes on (`scope.go`). The Codex refusal would therefore block a launch that Claude goes ahead with.
   - The refusal also names no next step, such as fixing or removing the named file. No row of `TestPrepare_UserSetup` asserts a way forward.

4. MEDIUM — D9, card 10: any walk or stat error other than a missing file refuses the launch, with no way forward named.
   - `unreadClaudeFiles` walks the whole checkout from its root, skipping only `.git`.
   - One directory the user cannot read (a root-owned container volume, a build cache) refuses every Codex launch in that worktree. The refusal does not say what to change.
   - Either skip an unreadable directory with a warning, as the pointers are advisory, or name the path and a fix in the refusal.
   - No planned row reaches this refusal: `TestPrepare_InstructionFiles` has no error row.

5. LOW — D9, card 4: a failed write of the import record is a render warning, and an unreadable record is a Warn log, both naming no next step.
   - In both states an edit to an imported file silently stops re-rendering `AGENTS.override.md` until the next full render.
   - The recovery that works is `lym heddle render`, since `recordImports` rewrites the record. Neither message names it.
   - Neither branch has a test row.

6. LOW — D8, card 8: a hook that fails or times out is logged at Warn with its cause, but the warning names no next step, such as fixing the hook or dropping the plugin's skills from the role.
   - The cause also reaches only `Omitted` and the log, so the operator gets no route to repair beyond reading the log.

No further tool calls beyond the two below.

- Grepped `internal/reedengine` for how the launch line reaches tmux: my context lacked whether the whole pane line or only the `-c` setting meets the per-argument cap.
- Read `internal/reedengine/doc.go:398-412`: it confirmed the line is written to a script and sourced, so the setting is the argument card 5 measures correctly.
