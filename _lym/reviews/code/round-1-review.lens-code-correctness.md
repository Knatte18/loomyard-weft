```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff `780b1db4..HEAD` (codexengine writable roots).

**Per function/branch, case run, verdict**

- `readCheckout` (`home.go:69-97`), plain clone: `FindRoot` returns `GitDir == CommonDir == <root>/.git`. Both `EvalSymlinks` calls succeed, so `gitDir == commonDir`. The `GitDir != CommonDir` branch is skipped and `mainRoot` is empty. Correct.
- `readCheckout`, linked worktree: `GitDir` is `<main>/.git/worktrees/wt` and `CommonDir` is `<main>/.git`. Both resolve and the main root is set. Correct.
- `readCheckout`, the new `EvalSymlinks` failure paths: `FindRoot` only returns success after `IsRepository` has `Stat`ed `GitDir/HEAD` and `CommonDir/objects` and `refs`, and `Read` has required the gitdir and commondir targets to exist. So `GitDir` and `CommonDir` exist at that moment, and the only way to reach the new failures is a race or a permissions change between the check and the resolve. No layout `FindRoot` accepts reaches the failure where the old code launched. Correct, not a defect.
- `readCheckout`, a git dir reached through a symlink: both are resolved, so the root is keyed as Codex keys it. Correct.
- `checkout.gitDirs` (`home.go:51-56`): plain clone gives one entry. A linked worktree gives the git dir and then the common dir. A worktree whose `commondir` resolves to the same dir as its git dir is deduped. Order and dedupe are correct for each layout.
- `configuredWritableRoots` (`config.go:130-147`):
  - A nil or empty list returns nil. Correct.
  - An empty string, `""`, makes `expandHomePath` refuse by key. Correct.
  - A relative value is refused by key. Correct.
  - `~/` expands under the home directory. A `~/` whose `UserHomeDir` fails is refused. Correct.
  - A path that does not exist returns `ErrNotExist` and is skipped. Correct.
  - A dangling symlink: `EvalSymlinks` returns an `lstat` or `readlink` `ErrNotExist`, so it is skipped. Correct.
  - A path that exists but is a regular file: `EvalSymlinks` succeeds and the file becomes a writable root. This is a minor gap; Codex receives a file as a root. Low impact (see Finding 1).
  - A permission error on `EvalSymlinks` refuses the launch with `codexengine: resolve codex.yaml's writable_roots "<root>": <err>` and no `way forward:` clause. Behaviour is correct, but this refusal lacks the way forward that every sibling codexengine refusal carries (see Finding 2).
  - `ENOTDIR`, a path component that is a file, is not `ErrNotExist`, so it refuses the launch where "a folder that does not exist is left out" would suggest a skip. Edge case (see Finding 1).
  - The doc claim "refusing an empty or relative value by its key" holds, because `expandHomePath` is the only gate and it names the key.
- `readyHome` (`codexengine.go:181-215`):
  - `configuredWritableRoots` runs after `checkCodexFolders` and before `prepareHome`, so a refusal leaves the lym home unmade. This matches the `RelativeWritableRoot` row asserting the home unmade. Correct.
  - Records told and resolvable gives `[records, gitDir, commonDir, configured...]`. Correct.
  - Records told but unresolvable: a Warn fires and the told path is kept, then the git dirs and configured roots follow. Matches the doc.
  - Records not told: `writableRoots` is nil, then the git dirs are appended. Correct.
  - `append(writableRoots, found.gitDirs()...)` appends to a slice whose backing array is the local `[]string{records}` or nil, so it aliases nothing shared. Correct.
  - `append(writableRoots, configured...)` is the same. Correct.
  - Duplicates between roots (configured equal to a git dir or the records dir) are not deduped. Harmless, since Codex takes the list as a set.
- `OpenLaunch` and `Prepare` callers: both take the new roots from `readyHome` unchanged. `tomlArray` is used as before. Correct.
- `ConfigSpec`, `writable_roots` as `CategoryGuard`: `LocalDenied` lists any non-None category, so a local override of `writable_roots` is refused. The commit-message claim holds. Correct.
- Template `writable_roots: [~/.cache/go-build]` unmarshals into `[]string`. Correct.

**Findings**

1. LOW, `design`, `internal/shuttleengine/codexengine/config.go:137-144`: `configuredWritableRoots` skips only `os.ErrNotExist`. A configured path that is a regular file becomes a writable root, and a path under a regular file (`ENOTDIR`) refuses the launch, though the doc says a folder that does not exist is left out. Evidence: `errors.Is(err, os.ErrNotExist)` is the only skip and nothing checks `isDir`.
2. NIT, `consistency`, `config.go:142`: the `EvalSymlinks` failure refusal for `writable_roots` has no `way forward:` clause, unlike the other refusals in `expandHomePath`, `validateEffort` and `checkCodexFolders`. `PATTERN-refusal-way-forward` binds only webster, shed and loom, so this is not a rule violation. Evidence: the message at `config.go:142`.

No correctness defect found in the git-dir roots, their order or their dedupe, the `readyHome` ordering, or the guard category.

**Tool calls:** none. Every read this lens needed was already in context (`home.go`, `config.go`, `codexengine.go`, `dotgit.go`, `prepare_test.go`).
