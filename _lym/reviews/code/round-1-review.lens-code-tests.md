```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

## Behaviour to test list

| Behaviour | Pinning test |
|---|---|
| Template defaults: README.md for a beam, nothing for prime or task, `render_markdown` true | `TestLoadConfig/TemplateDefaults` |
| A hub file sets both keys, with a `{beam}` entry accepted | `TestLoadConfig/HubFileSetsBothKeys` |
| A local override sets `render_markdown` | `TestLoadConfig/LocalOverrideSetsRenderMarkdown` |
| A local override sets `open_files` | none in `ideconfig`. The configreg `wantLocalKeys` row pins that the key is open. |
| Empty, absolute and climbing entries are refused, with `ErrInvalid` naming the key | `TestLoadConfig/{AbsoluteEntryRefused,ClimbingEntryRefused,EmptyEntryRefused}` |
| Windows slash handling in validation | none. The tests do not run on Windows. |
| `ide` is registered hub-wide with both keys local | `TestModules_SeedOnlyAndHubWideFlags` and `TestModules_EveryTemplateKeyClassified` |
| `ideconfig` is in the degrading call-site set | the existing `configstrictness_test.go` pin, updated in the diff |
| A beam spawn opens its listed files in order, with `{beam}` resolved (also inside a longer entry) and an absent entry skipped | `checkBeamOpensBeamWorkspace`: README.md, `absent.md`, `docs/{beam}.md` |
| A beam workspace carries the `*.md` preview association | `checkBeamOpensBeamWorkspace`, `readWorkspaceLook(...).Associations` |
| A task spawn opens no file under the template | `checkTaskSlugOpensBareFolder` |
| `*.md` association set on a fresh file | `TestWriteConfigOwnsMarkdownPreviewEntry/FreshFileRendered` |
| `*.md` association set on an existing file, keeping other associations | `.../RenderKeepsOtherAssociations` |
| `render_markdown` off leaves a fresh file with no key | `.../FreshFileNotRendered` |
| Off removes the preview entry and drops the emptied map | `.../OffRemovesPreviewAndEmptyMap` |
| Off leaves an operator's own `*.md` association | `.../OffKeepsOperatorsMarkdownEditor` |
| `WriteConfig` and `CodeLauncher` signature changes | compile-checked by the updated callers in `idecli/cli_test.go`, `menu_test.go` and the integration test |
| `{beam}` resolved for a beam | covered by the beam spawn test above |
| `{beam}` resolved for the prime | none |
| `{beam}` resolved for a task, through `taskBeam` (task forked from the prime or from a beam) | none |
| `taskBeam` Warn and `""` on a prime or parent resolution failure | none |
| Prime spawn opens the `prime` list | none. No prime test sets `ide.yaml` or inspects the launcher's files. |
| `OpenTask` passes the task's files to the launcher | none. `checkWindowCommandFollowsALaunch` only checks the launcher's error and the window command. |
| A non-regular entry (a directory) is skipped | none. Only an absent file is exercised. |
| `{beam}` with no resolvable beam is skipped and logged | none |
| `seed` with `primeResolved` false still gets the task path and files | none in the diff. The restructured `primeResolved` branch has no new row. |
| An `ide.yaml` that fails to load fails `Spawn`, `Seed` and `OpenTask` | none. `ideconfig` pins the refusal in `LoadConfig`, but no `ideengine` test asserts that the verb fails and writes nothing. |
| `render_markdown: false` threaded through `seed` into the worktree `settings.json` | none at the engine level. `vscode` pins the write itself. |
| `*.md` association in an existing JSONC `settings.json` with comments | none. The new table rows use plain JSON. |
| `*.md` association spliced into the prime workspace file | none. Only the beam workspace is asserted. |
| `vscode.Launch` puts the target before the files on Linux and Windows | none. Both launchers exec `code` or `cmd /c code`, and no test or seam covers the argument order. |

## Findings

1. **MEDIUM, class consistency.** `{beam}` resolution for a task, through `taskBeam`, has no test.
   - Location: `internal/ideengine/spawn.go` `taskBeam`, `filesToOpen`.
   - The summary claims `{beam}` resolves to "a task's parent's, or the prime's name for the prime and a task forked from it".
   - The template's task list is empty, and `filesToOpen` runs `beam` lazily, so `taskBeam` never runs in any test.
   - The only task assertion is "no file under the template" at `checkTaskSlugOpensBareFolder` (`spawn_hub_integration_test.go`, diff hunk near line 234).
   - Add a row with `task: ["{beam}.md"]` for a task forked from the prime and one forked from a beam.

2. **MEDIUM, class consistency.** `OpenTask` passing files to the launcher is uncovered.
   - Location: `spawn.go` `OpenTask`.
   - Evidence: `checkWindowCommandFollowsALaunch` (diff hunk near line 377) only swaps in a launcher that returns an error.
   - A regression that drops the files, or ignores a load failure, still passes.

3. **MEDIUM, class consistency.** The summary says an `ide.yaml` that fails to load fails `spawn`, `seed` and `open`, but no `ideengine` test asserts it.
   - Location: `spawn.go` `seed` and `OpenTask`.
   - Also unasserted: that `seed` writes nothing on that failure, because the load now comes before `writeWorktreeConfig`.
   - This is the behaviour change the focus directive questions.

4. **MEDIUM, class consistency.** The prime path has no coverage.
   - Location: `spawn.go` `seed`, the `ide.OpenFiles.Prime` assignment.
   - No test sets a `prime` list or checks the files passed for the prime.
   - The `seed` rewrite also changed the beam path (prime list assigned, then overwritten); no test covers it.

5. **LOW, class consistency.** `filesToOpen` branches with no row:
   - a directory or other non-regular entry skipped;
   - the beam-unresolvable skip, where `beam()` answers `""`;
   - the `primeResolved == false` task path.
   - Location: `spawn.go` `filesToOpen` and `seed`.
   - These fit as rows in the existing beam and task integration checks.

6. **LOW, class consistency.** `vscode.Launch` argument order is unpinned on both platforms.
   - Location: `internal/vscode/launch_linux.go` and `launch_windows.go`.
   - The two argument builders are inline and no seam or test covers them.
   - Fixing it would need extracting an argument-builder function, so it is a cheap-to-skip gap.

7. **LOW, class consistency.** `render_markdown: false` through `seed`, and the association in a JSONC existing file or a prime workspace, have no engine-level row.
   - Location: `spawn.go` `seed`, `internal/vscode/config.go` `applyMarkdownPreview`.
   - Row-level behaviour in `vscode` is pinned, so this is a wiring gap only.

I found no test economy violations. Each new `ideconfig`, `vscode` and `configreg` row covers a distinct behaviour.

## Tool calls

No tool calls were made. The diff and tests were already in context.
