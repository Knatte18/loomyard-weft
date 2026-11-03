```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set for twins or siblings left inconsistent (fix-consistency lens).

**Findings**

1. **LOW** — `internal/battencli/cli.go:69` and `wire.go:263-265`: the docs still say "every batten verb" meets a stray directory as a gone pair, and that `taskWorktreePresent` is batten's "one answer". Two stat-only presence checks in the package remain.
   - `beamland.go:140`, `pairPresent`, uses `os.Stat` plus `IsDir`. It reads a stray directory as present.
   - `overview.go:290` uses a bare stat to decide whether to drop the pair from `r.pairs`. It also keeps a stray directory as a pair.
   - The review's rejected list treats `beamfork.go` as already covered. It named neither of these two sites.
   - Both are in-scope siblings of the claim the F1 fix kept in the docs, because it chose the "make the code match" option rather than the "soften the docs" one. If they are intended exceptions, the docs overclaim. If not, the twin of the F1 change was left as it was.
   - I did not check whether `pairPresent` is the pair check behind the beam-land member listing, so I can't say whether a stray directory there causes a visible misreading.

2. **LOW** — `wire.go` doc of `taskWorktreeLocation` (lines ~200-203): it says "An absent pair is refused by name". That now also covers a stray directory. The sentence is still true but silent about the new case. `presentTaskWorktreePath`'s doc says "refusing an absent one by name", and its callers now also refuse an unresolvable directory. The `taskWorktreeLocator` doc was updated for the cached stat-only branch, so the locator docs agree with their code.

**Checked and consistent**
- The locator's cached branch stays stat-only, as `stepTaskWorktreeSeams_RunGitOnlyUntilTheFirstResolution` requires, and the gate passed.
- `absentTaskWorktreeRefusal` keeps the refusal text byte-identical.
- The F2 subtest follows the sibling subtests' pattern: same seam, same `refuse` arguments, with only the status changed.

**Further tool calls (2)**
- I ran a grep for `os.Stat(fabricengine.WorktreePath`, `taskWorktreePresent` and the doc phrases, plus the `cli.go:62-74` slice. My context lacked the other stat-only sites and the `cli.go` doc.
- I read `overview.go:280-296` and `beamland.go:130-150`, to see how those two stat sites use the result.
