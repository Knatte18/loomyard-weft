```yaml
lens: code-correctness
outcome: reported
```

**Task:** correctness lens over the code the diff changes. For each function or branch: the case I ran, and whether it came out correct.

## Per-branch list

- `seed`, ide.yaml load before any write: `ide.yaml` with an absolute entry gives an error before `writeWorktreeConfig`, so nothing is written. Correct.
- `seed`, restructured prime branch, prime resolved and `slug == primeName`: `isPrime` is true, the hub workspace is regenerated, launch path and window directory are as before, files come from the Prime list with `{beam}` = prime name. Correct.
- `seed`, beam (`primeResolved`, `isBeam` true): `openFiles` is first set from the Prime list, then overwritten by the Beam list. Launch path and window directory are as before. Correct, if redundant.
- `seed`, prime unresolved: `isPrime` is false and `!primeResolved` is true, so it returns early at the task branch. Launch path and window directory match the old early return, and no workspace is regenerated. `PrimeName` is called a second time inside `taskBeam`, but only when an entry names `{beam}`. Correct.
- `seed`, `isBeam` fails to read the origin record (returns false): the slug takes the task branch and the Task list. Launch path is unchanged from before. Correct.
- `filesToOpen`: empty `entries` gives nil. `.` and `docs/` are directories and are skipped. `{beam}` with an empty beam name is skipped. A placeholder inside a longer entry (`docs/{beam}.md`) is replaced and then `Stat`ed. `beam()` runs once. Correct. Placeholder replacement happens after the climb check at load; slugs can't hold separators, so this holds.
- `taskBeam`: a task forked from a beam returns the parent slug, and one forked from the prime returns `primeName`. A `PrimeName` or `parentOf` error returns `""` and the entry is skipped with a Warn. Correct.
- `OpenTask`: a load failure returns before the launch and writes nothing. Correct.
- `ideconfig.LoadConfig`: an empty or null entry gives `IsLocal("")` false, so it is refused. `a/../b` is allowed. `..` is refused. An absolute path is refused. A missing hub file resolves the template. Correct.
- `applyMarkdownPreview`, render on: a fresh settings file gets the map, and an existing map keeps its other entries. See finding 1 for the array form.
- `applyMarkdownPreview`, render off: the entry is removed only while it names the preview, and an emptied map is dropped. A nil map read is safe. Correct.
- `Launch` on Linux and Windows: the argument order is target then files on both. An absolute path starting with `/` or a drive letter cannot be read as a flag. Windows `cmd /c code ...` starts with `code`, not a quote, so `cmd`'s quote-stripping does not apply. Correct.

## Findings

1. **LOW, design.** `internal/vscode/config.go`, `applyMarkdownPreview`: the type assertion `settings[editorAssociationsKey].(map[string]any)` fails for the array form of `workbench.editorAssociations`, `[{"viewType": …, "filenamePattern": …}]`. As far as I know, VS Code accepts that form. With render on, `associations` is then nil, a fresh map replaces the key, and the operator's array is silently destroyed. The doc comment promises "keeping every other association." Evidence: `config.go` lines 556-564 of the diff. Fix: leave a non-map value alone, or convert it.

2. **LOW, design.** `internal/ideengine/spawn.go` `seed`, `OpenTask`: both load `ide.yaml` over `fabricengine.ConfigDirs(l)`, the caller's location, not the opened worktree's. `lym ide spawn <slug>` run from another worktree therefore ignores the target's local override. Both `open_files` and `render_markdown` are declared open to the local layer for exactly this reason. The summary admits "caller's config stack", but the override is silently inert for the worktree being opened. Evidence: `spawn.go` diff lines 335 and 382.

3. **LOW, design.** `internal/idecli/cli.go:109-111` with `spawn.go` `OpenTask`: `lym ide open <slug>` accepts any hub slug, including a beam or the prime. `OpenTask` always uses the Task list and `taskBeam`. For a beam, `{beam}` resolves to the beam's parent, which is the prime's name or `""`, not the beam itself. The verb is documented for task worktrees, so this is mild.

4. **LOW, design.** `taskBeam`: a task forked from a non-beam task resolves `{beam}` to that parent task's slug, not a beam. The doc says "the pair it was forked from", so this matches the doc but not the placeholder's name. Evidence: `spawn.go` diff lines 422-441.

## Tool calls

1. Read `internal/idecli/cli.go` lines 60-130: the manifest lacked who calls `OpenTask` and with which slugs (finding 3).
