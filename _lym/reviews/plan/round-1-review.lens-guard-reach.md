```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For every guard, refusal, branch or test row the cards add, I traced whether a real entry point reaches it.

**Per guard, reachable or not**

- **Card 1, `Expand` read error on `path`:** reachable. heddle's `fileText` and card 7 call it on a CLAUDE.md that can't be read.
- **Card 1, other branches:** the import rows (code span, fenced block, missing target, hop limit, cycle, `~/`) are all reachable from file contents.
- **Card 2, `Show(agents-prompt)` refusal:** reachable. `showCmd` passes `args[0]` to `Show` with no check of its own (`internal/heddlecli/verbs.go:358-363`).
- **Card 2, `promptTargetsFor` skipping a shared row and `passagesOf` reaching through `RendersInto`:** reachable through `Passages`.
- **Card 3, no-seam case of `NotifyWorktreeMadeAt` returning "":** reachable (test processes, or no seam installed).
- **Card 3, unresolved-path warning:** reachable, though rarely. In production `w.Path` comes from `CodeWorktrees` and `OpenCodeWorktree` has already succeeded, so only a race or a broken anchor marker reaches it. The test reaches it with a path in no worktree.
- **Card 3, `FailingSeam` row:** reachable with a fake seam.
- **Card 3, lock check:** none of the `Ensure` call sites (`battencli/wire.go:351`, `beamcli/cli.go:511`, `adoptcli/check.go:476`, `fabriccli/clone.go:147`) holds the Bolt write lock that the render's read takes, so the render does not stall there.
- **Card 4, `RecordUnwritable` warning:** reachable (a directory sits at the record's path).
- **Card 4, `recordedImports` absent or malformed record:** reachable.
- **Card 4, empty `Root` writes no record:** NOT reachable in production (finding 3).
- **Card 5, refusal past 128 KiB:** reachable. heddle puts no size limit on an entry, and `StandingPassages` passes texts through unchanged. The boundary is wrong, though (finding 1).
- **Card 5, empty set omits the key:** reachable when no hub is told and no notice or passages exist. It is unreachable in a hub, where the creel sentence is always present, but the omission is by design.
- **Card 6, `max_subagent_threads` &lt; 1 refusal:** reachable. The key is `CategoryNone`, so the local, worktree and hub layers can all set 0. It also fires through the registry's config validator.
- **Card 6, effort codex.yaml doesn't list returns "" with a warning:** reachable (agents.yaml `xhigh`).
- **Card 6, no role effort:** reachable.
- **Card 6, `codexCommand` cap ≤ 0 branch:** reachable only from `OpenLaunch`, as the card intends.
- **Card 7, global or above-root file read error refuses:** reachable (a non-root unreadable file, or a directory named CLAUDE.md). Its way forward does not fit every case that reaches it (finding 4).
- **Card 7, malformed plugins file, missing `installPath`, unreadable SKILL.md give warnings:** reachable.
- **Card 7, `claudeConfigDir` failure:** reachable (HOME and CLAUDE_CONFIG_DIR both unset), but the card prescribes no handling (finding 5).
- **Card 8, malformed `hooks.json` gives an `Omitted` entry:** reachable.
- **Card 8, matcher filter:** reachable.
- **Card 8, hook error or timeout gives `Omitted`:** reachable.
- **Card 8, process-group kill on timeout:** reachable (`sleep` forked by `sh`).
- **Card 8, nil `RunHook` gives "not run":** never reached in production, since the registry always passes `RunPluginHook`. The board entry's Exports license it as a seam for launch-manifest-parity, so it is not a finding.
- **Card 9, source error refuses:** reachable through card 7's read-error refusal.
- **Card 9, `plugin_hook_timeout` not a positive duration:** reachable (`soon`, `0s`).
- **Card 9, omitted hooks and source warnings reach `c.warn`:** reachable.
- **Card 9, nil `SandboxProbe` keeps `runSandboxProbe`:** reachable only in production wiring. The card states this.
- **Card 9, `TestNew` reaching `Prepare` with no probe:** reachable. `launchSandboxMode` returns early on `danger-full-access` (`internal/shuttleengine/codexengine/sandbox.go:71-73`), and the worktree layer may set the machine key.
- **Card 10, `projectDocMaxBytes` stat error:** a raw I/O error, so exempt from the reaching-test rule.
- **Card 10, sum below 32 KiB returns the default:** reachable.
- **Card 10, unreadable directory gives a warning:** reachable for a non-root user. The row skips as root.
- **Card 10, `AGENTS.override.md` exception at the working directory:** reachable, but it checks the wrong condition (finding 2).

**Findings**

1. **MEDIUM, design. Card 5 (`maxArgumentBytes`, the `AtTheArgumentCap` row).**
   - The card lets a setting of exactly 131072 bytes go ahead, and pins that in a test.
   - Linux's `copy_strings` counts the terminating NUL. It checks `strnlen_user(str, MAX_ARG_STRLEN) &lt;= MAX_ARG_STRLEN`, so the longest argument `execve` accepts is 131071 bytes.
   - A 131072-byte setting therefore fails at exec with E2BIG, which is the state the refusal exists to prevent.
   - The guard should refuse at `&gt;= maxArgumentBytes`, and the boundary row should expect 131071 bytes to go ahead and 131072 to be refused.
   - The launch reaches argv through the pane's sourced launch script (`internal/reedengine/spawn.go:112-119`), so exec's limit applies to that one argument.
   - Evidence: `_lym/plan/05-codex-developer-instructions.md:47-49,67`.

2. **LOW, consistency. Card 10 (`Codex.unreadClaudeFiles`, the exception at the working directory).**
   - The exception skips the anchor's CLAUDE.md beside an `AGENTS.md` whenever `AGENTS.override.md` exists there, on the grounds that heddle's render carries both files.
   - `Render` leaves an override file without heddle's `Marker` untouched, with a warning (`internal/heddleengine/render.go:149-151`). So a repository's own `AGENTS.override.md` can exist without carrying CLAUDE.md.
   - In that case Codex reads only the override, and the CLAUDE.md is neither carried nor pointed to.
   - The check should test for heddle's `Marker`, not only for the file's existence.
   - Evidence: `_lym/plan/10-codex-instruction-files.md:47`.

3. **LOW, design. Card 4 (`Renderer.Render`, the empty-`Root` branch, and the `NoRoot` test row).**
   - After card 4, both production builders of `RenderRequest` set `Root`: `requestOf` (`internal/heddleengine/seam.go:80`) and `heddlecli`'s `renderInto` (`internal/heddlecli/verbs.go:553`).
   - So "an empty `Root` writes no record" guards a state no production entry point produces.
   - Per the card rules, the branch should be dropped along with its row, or written as a fail-loud wiring guard.
   - Evidence: `_lym/plan/04-codex-file-carries-both.md:40,46,61`.

4. **LOW, consistency. Card 7 (`UserSetupSource.UserSetup`, the read-error refusal for above-root files).**
   - The walk above `RepoRoot` reaches machine directories, as the card's own test note admits ("the walk above the root also reaches the machine's own directories").
   - An unreadable CLAUDE.md the user does not own, such as a root-owned `/home/CLAUDE.md`, therefore refuses every Codex launch.
   - The prescribed way forward ("fix its permissions or remove it") is not open to a user who does not own the file.
   - Evidence: `_lym/plan/07-claude-user-setup-source.md:49-52,74`.

5. **LOW, design. Card 7 (`NewUserSetupSource` and `UserSetup`, failure to resolve the config directory).**
   - In production `claudeDir` is empty and resolves through `claudeConfigDir`, which returns an error when HOME cannot be resolved (`internal/shuttleengine/claudeengine/scope.go:140-148`).
   - The card prescribes no outcome for that branch: refuse, or warn and add nothing, as `sessionScopeFor` does.
   - Evidence: `_lym/plan/07-claude-user-setup-source.md:47,52`.

**Tool calls**

1. `heddlecli/verbs.go` `showCmd`, plus a grep for production `RenderRequest{` builders: to check whether the `Show` refusal is pre-empted and whether any builder leaves `Root` empty.
2. A grep of the lock calls around each `hubreconcile.Ensure` call site: to check whether card 3's render can block on the Bolt lock.
3. `codexengine/sandbox.go` `launchSandboxMode`: to confirm card 9's `TestNew` row reaches `Prepare` without a probe.
