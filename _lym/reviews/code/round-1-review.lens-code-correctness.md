```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `tapestryrename` program, the renamed recipe constant and the retired-row map.

**List (function or branch, case run, verdict)**

- `substitute` / `matchAt` (`tables.go`): `heirloom run` stays, because `m` before `l` is alphanumeric and `loom run` starts with a lowercase letter. Correct. `loom runs` picks the longer key. Correct. `loom running` stays, because the next byte is a lowercase letter. Correct. Empty text returns empty. Correct.
- `tokenStart` `lym ` guard: `` `lym loom run` `` is kept. Correct. A `lym` and `loom` split by a newline or two spaces is not guarded. That is a latent gap, not an observed defect.
- `substitutionKeys` / `pathSubstitutions` for `registry_test.go`: the longer path key wins over the `LoomPreflight` identifier key at the same position. The output is not re-scanned, so the two path keys fire once each. The resulting list is sorted, and a second run changes nothing. Correct.
- `applyMoves`:
  - Source present, target absent: moves. Correct.
  - Source absent, target present: skipped, so a second run is idempotent. Correct.
  - Both present: returns an error. Correct.
  - Both absent: skipped silently. Acceptable.
  - Under dry-run nothing is moved, and `substituteTree` then reads the old paths. Correct.
- `substituteTree`:
  - A Go file with a failing `format.Source` returns the error after earlier files were already written. That is acceptable for a one-shot tool.
  - The file mode is kept. Correct.
  - Symlinks are skipped. Correct.
  - Excluded directories are skipped via `SkipDir`. Correct.
- `excluded`: the entry `.git/` only matches a directory. In a worktree `.git` is a regular file, so `.git` is not excluded and is read and rewritten if a key matches. Its `gitdir:` path holds no key today, so nothing changes. Finding F1 (LOW).
- `RecipeNames` (`internal/shedrun/seed.go`): the order batten, darn, pawl, tapestry is sorted. Correct. `ValidateRecipe("loom")` fails with `unknown recipe "loom"; available recipes: [batten darn pawl tapestry]`. D6 holds. There is no alias, legacy constant or decode mapping for `loom`.
- `registry` map (`internal/shedrecipe/registry.go`): the key `TapestryPreflight` is paired with `tapestryPreflightEntry`. Only the whitespace alignment changed. Correct.
- `boardengine` `defaultRecipe` and `recipeCell`: an empty recipe renders `tapestry`. Correct. The `normalizeTask` refusal text now reads `tapestry or darn`. Correct.
- `TapestryRetiredRows` (`internal/loomrecipe/loomrecipe.go:17-19`): it maps only `Merge-In` to `Parent-Align`. It has no `Loom-Preflight` to `Tapestry-Preflight` entry. A status file parked on `Loom-Preflight` therefore has no successor, and `peers.go:88` and `arm.go:197` find no rename for it. The decision record's Constraints require this entry ("Row names are durable resume identities, so the Loom-Preflight rename goes with a retired-row entry"). The focus directive also says the entry is still required. Finding F2 (MEDIUM).
- `valueSubstitutions` key `keys := loom["loom"]`: it is rewritten together with `loom, err := profiles.Settings(...)` as `tapestry`. Any other use of the local `loom` in that file would be left behind. Whether the file compiles is the gate's concern, so I raise nothing.

**Findings**

- F1, LOW, design, `tools/tapestryrename/tables.go:107` and `main.go:84`: `.git/` excludes only a `.git` directory. In a worktree `.git` is a file, so it is not excluded. Fix: add `.git` as an exact-match exclusion, for example `".git"` with a path-equality check.
- F2, MEDIUM, design, `internal/loomrecipe/loomrecipe.go:17-19` (also `internal/loomrecipe/doc.go:27`): `TapestryRetiredRows` lacks `"Loom-Preflight": loomshed.NameTapestryPreflight`. The Constraints require it, and nothing in the diff carries it. Fix: add the entry and a row in `TestRenameRetiredRows` (`internal/loomcli/wiring_darn_test.go`), and update the doc comment. This is the "retired-row entry the Constraints still require" that the focus directive names. The parent's withdrawal covers an alias, `decodeSeed` mapping and cross-hub migration, not this entry.

**Tool calls**
1. `sed`/`grep` over `internal/loomrecipe/loomrecipe.go`, `ValidateRecipe` in `seed.go`, and `git grep Loom-Preflight`. My context did not hold `TapestryRetiredRows`' body or `ValidateRecipe`'s refusal text. The grep found no remaining `Loom-Preflight` in `internal` or `contracts`.
