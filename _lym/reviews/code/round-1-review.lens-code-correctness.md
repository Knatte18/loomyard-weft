```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, listing each function or branch, the case I ran and the result.

**Per-function list (case → result)**

- `developerInstructions` (`developerinstructions.go:445`): a setting of exactly 131071 bytes goes ahead, one of 131072 is refused. The refusal names the size, the cap, the largest item (first wins on a tie) and its way forward. The item list always holds the creel head, so the empty-set branch is unreachable from `Prepare`. Correct.
- `codexCommand` (`command.go:107-122`): an empty instruction string, a zero thread cap and a zero doc cap each omit their `-c`. The quoting goes through `sh.Quote`. Correct.
- Developer-instructions item order in `Prepare` (`codexengine.go:157-178`): creel head, subagent passage, standing passages, instruction files, pointers, skill listing, plugin contexts. This matches the order `claudeengine.go:260-266` writes, since the subagent passage is prepended to the passages. It differs from the record's parenthetical order, but the plan's order matches Claude's file. Correct.
- `claudeimports.Expand` (`claudeimports.go:34`):
  - The `f0` to `f6` chain imports through `f5` and leaves `@f6.md`. Correct.
  - A cycle stops at the repeated file. Correct.
  - A missing file, a directory, an import in a code span or an import in a fence stays as written. Correct.
  - A fenced block closed by a line carrying an info string counts as closed (`:64`). Real CommonMark does not close a fence on such a line. LOW, noted only.
- `RunPluginHook` (`pluginhook.go:388`):
  - A timeout kills the process group, and the cancel race still reports a timeout.
  - A non-zero exit returns an error carrying stderr.
  - Stdin is a `bytes.Reader`.
  - Correct.
- `pluginSessionStartContexts` (`pluginhook.go:325`):
  - A hooks-file error becomes one omitted entry.
  - A nil runner omits each hook as not run.
  - The timeout falls back to the request's value.
  - An off plugin runs nothing.
  - Correct.
- `UserSetupSource.UserSetup` when the config dir cannot be resolved (`usersetup.go:538`): the global file, plugins, user skills and plugin contexts are dropped. Project skills and the above-root files are kept, with one warning and no error. Correct.
- `installedPluginRoots` (`usersetup.go:666`): a plugin with a user-scope install that has an empty `installPath` hits the `break` at `:674` and ignores a later valid project-scope install of the same plugin. Edge case, LOW.
- `heddleengine.recordImports` (`importrecord.go:43`): an import outside the root is dropped, and an empty list clears a dropped import. Both `RenderRequest` constructors set `Root` (`seam.go:84`, `verbs.go:554`). Correct.
- `Seam.TookIn` (`seam.go`): a changed import path selects only the `codex-file` target. Correct.
- `NotifyWorktreeMadeAt` and the `walk.go:327` caller: `fireWorktreeEvent` logs failures through `seamWarning`, so discarding the return with `_ =` matches the comment. Correct.
- `projectDocMaxBytes` (`instructionfiles.go:523`):
  - It sums the first existing file of override, `AGENTS.md`, `CLAUDE.md` per directory from the root down, floored at 32 KiB.
  - A stat error other than not-exist refuses the launch.
  - Correct for the cases run.
- `unreadClaudeFiles` and `claudeFileIsUnread`: see F2 and F3 below.

**Findings**

F1 — MEDIUM, design. `internal/shuttleengine/codexengine/codexengine.go:123-124`. `Prepare` passes the unresolved `c.dirs.Anchor` as `WorkDir` but the symlink-resolved `found.root` as `RepoRoot`. `readCheckout` resolves the root (`home.go:78`), and `Prepare` itself resolves `workDir` separately for the doc cap (`codexengine.go:113`).
- Where the anchor path contains a symlink, `skillListing`'s upward walk (`claudeengine/usersetup.go:744-751`) never meets `repoRoot`. It then climbs to `/`, listing `.claude/skills` of every ancestor, including the home directory's.
- Where the anchor is a symlink path, the `projectPath` match in `installedPluginRoots` (`:668`) and the hook `cwd` and `CLAUDE_PROJECT_DIR` use the unresolved form.
- Fix: pass the resolved `workDir` as `WorkDir`, or resolve both inside `UserSetup`.

F2 — LOW, design. `internal/shuttleengine/codexengine/instructionfiles.go:600-601`. A `CLAUDE.md` with imports at `workDir`, with a heddle-rendered `AGENTS.override.md` but no `AGENTS.md`, falls through to the import branch and is named as unread. The override already carries it expanded, and the `heddleRendered` exception at `:595` applies only when an `AGENTS.md` exists. No test row covers this. Fix: apply the `heddleRendered` exemption at `workDir` before the import branch.

F3 — LOW, design. `internal/shuttleengine/codexengine/instructionfiles.go:565`. `unreadClaudeFiles` walks the whole checkout on every `Prepare`. It does not skip git-ignored trees such as `node_modules`, `.scratch` or build output. That costs time on large trees and lists vendored `CLAUDE.md` files as pointers. Fix: skip directories that git ignores, or a closed list of dependency and scratch directories.

**Tool calls (4 Bash, 1 Read)**
1. `grep` of `readCheckout` in `codexengine/home.go`: to see whether `found.root` is symlink-resolved (it is).
2. `grep` for the `claudeengine` subagent/creel/passage symbols: to find the Claude prompt-order site.
3. `grep` of `fireWorktreeEvent` and `seamWarning` in `fabricengine/worktreeseam.go`: to check that a failed render is logged.
4. `grep` of non-test `RenderRequest{` constructors: to check that every one sets `Root`.
5. Read of `claudeengine.go:255-285`: to confirm the order Claude's appended prompt holds.
