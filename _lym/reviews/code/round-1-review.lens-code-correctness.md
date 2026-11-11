```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness for the `removeIgnoredTaskPath` diff, reported as per-function cases, then findings.

**Cases run**
- `add.go:338-348` (Add's task-path branch)
  - Input: leftover of only ignored output, with a failure later in Add (no remote, `ErrNotLive`, `ErrRecordsWorktreeNotPair`, detached HEAD). **Incorrect:** the directory is already deleted when Add then refuses (F1).
  - Input: `os.Stat` fails with something other than not-exist, such as permission denied. `removeIgnoredTaskPath`'s `Lstat` then likely fails too, so Add returns `read &lt;path&gt;: …` instead of `*ErrTaskPathTaken`. Misreported, though minor (F2).
- `removeIgnoredTaskPath` (`destroy.go:1388-1419`)
  - Symlink at the target: `Lstat` shows a non-directory, so it returns false and Add gives `ErrTaskPathTaken`. Correct.
  - Regular file at the target: false, then `ErrTaskPathTaken`. Correct.
  - Registered linked worktree: false. Correct. A path-spelling mismatch is backstopped by the `.git` check.
  - `List` fails: returns a hard error instead of `ErrTaskPathTaken` (F2).
  - `check-ignore` fails: warns, keeps the path, returns false. Correct.
  - Delete fails: error wrapping `ErrStrayPathKept` with a way forward, path kept. Correct.
  - Delete succeeds: `KindPathRemoved` is appended, and the deferred snapshot in Add carries it on error returns too. Correct.
- `filesUnderTaskPath` (`destroy.go:1425-1448`)
  - Empty directory: no files and `holdsRepository` false, so `ignoresEveryPath` returns true and the directory is deleted. Matches the test.
  - `.git` at any depth, file or directory: `SkipAll` returns nil and `holdsRepository` is true, so the path is kept. Correct.
  - Symlink to a directory: `WalkDir` does not follow it. It is listed as a file, and git does not treat a symlink as a directory, so a `obj/` rule does not ignore it and the path is kept. Conservative and correct. A symlink to a directory that a file-level rule does ignore is deleted as a link, not through it, by the `os.Root` removal. Correct.
  - Unreadable subdirectory: `walkErr` becomes a hard error without a way forward (F2).
- `ignoresEveryPath` (`destroy.go:1454-1478`)
  - Empty list: true. Correct.
  - Exit 1 (none ignored): false. Correct.
  - Exit 128: wrapped error, so the caller keeps the path. Correct.
  - Partial ignore, exit 0: the output set misses a path, so false. Correct.
  - `-z` output: the trailing NUL leaves an empty-string entry in the map, which is harmless.
  - Path with leading `:` such as `:!x`: `check-ignore` parses operands as pathspecs, so the name is misread. It is then not reported as ignored and the path is kept, which is safe (F4, folded into F3's fix).

**Findings**

**F1. MEDIUM, design.** `add.go:338-348`
- The leftover is deleted as a mutation before Add's later pre-flight refusals. The later checks still to run are:
  - no remote and missing weft repo (`add.go:350-364`);
  - `ErrRecordsWorktreeNotPair` (`add.go:369-384`);
  - the adopt-local checked-out guard (`add.go:388`);
  - detached or unborn HEAD (`add.go:410`);
  - `ErrNotLive` (`add.go:444`, `add.go:461`);
  - `judgeWarpLeftover` and probe errors (`add.go:435`, `add.go:451`);
  - the anchor refusal (`add.go:465`).
- A refused Add therefore still destroys the directory. That contradicts the Add doc: "refused before any mutation", and gc and ownership "once its pre-flight refusals have passed". The comment at `add.go:470` says the same.
- The `AdoptOnly` re-adopt path, which can end in `ErrNotLive`, is affected too.
- Fix: judge the path early, but delete it only after the `add.go:470` marker, just before `createGitWorktree` at `add.go:492`. Keep `ErrTaskPathTaken` at the original position for the not-removable case.

**F2. LOW, consistency.** `destroy.go:1389-1412`
- Operational failures of the new branch (`Lstat` after `Stat`, `List`, `WalkDir`) return a bare wrapped error. The old path always returned `*ErrTaskPathTaken`, which `battencli/wire.go:393` and `readoptstop.go:31` turn into guidance for the parent.
- Fix: have the helper keep the path and return false, logging the cause as it does for `check-ignore` failures, so Add still returns `ErrTaskPathTaken`.

**F3. LOW, design.** `destroy.go:1454-1478`
- Ownership is judged as "ignored in the checkout of `l`", with the leftover's relative paths treated as repo-root paths. It is not judged against the new pair's own branch, which may carry other `.gitignore` files.
- Anything git ignores is deleted without a way to review it, including a user's `.env` or `.idea` content, not only build output. This is probably intended, since the entry asks for git-ignored output, so it is a risk note and not a defect.
- Fix: none required; optionally pass `--` before the paths to stop option and pathspec misparsing (F4).

**F4. NIT, design.** `destroy.go:1459`
- The `check-ignore` operands are parsed as pathspecs. A name starting with `:` is misread, and the outcome is only that the path is kept.

**Tool calls:** none. The context already held every function and git's `check-ignore` contract.
