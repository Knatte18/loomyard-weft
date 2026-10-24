---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/cluster.go:resolveShellPath
    summary: A .. under a ~ or $ anchor is clamped at the anchor, so a path above a recorded anchored repository resolves inside it and is wrongly excused
    fix: In resolveShellPath, when cleaning the part after a ~ or $ anchor would let .. climb above the anchor, return a result that no recorded repository can match (for example keep the unresolved .. element in the string) instead of clamping. Add a row to TestFirstMutatingGitCommandScopesToForkRepositories, git init "$S" then git -C "$S/../target" commit -m x, expecting the second command to be returned.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/cluster.go:record
    summary: A relative git init path that names the starting directory through .. and its basename is recorded as a fork repository
    fix: In record, refuse to record a relative created path whose first element is .., so git init ../name never creates a recorded repository. Add a row, git init ../x then git -C ../x commit -m x, expecting the second command to be returned.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/burlerengine/cluster_test.go:TestFirstMutatingGitCommandScopesToForkRepositories
    summary: Behaviours the diff adds have no row, namely the -- terminator and the valued options of git init and git worktree add, a trailing slash on a created path, and an anchored cd restoring a known directory after a bare cd
    fix: Add rows to the existing table. git init -- /tmp/r then git -C /tmp/r commit -m x, passing. git -C /tmp/r worktree add -b br /tmp/w then git -C /tmp/w commit -m x, passing. git init /tmp/r/ then git -C /tmp/r commit -m x, passing. cd && cd /tmp/r && git commit -m x after git init /tmp/r in the same command, passing.
  - id: F4
    severity: NIT
    class: consistency
    origin: handler
    location: internal/burlerengine/doc.go:305
    summary: The fork-discipline paragraph lists what never passes but omits the bound that a git init of the starting directory or an ancestor creates no recorded repository
    fix: In the paragraph in doc.go that begins a mutating invocation also passes, add a clause after the git push, --git-dir, --work-tree list saying that a git init naming the starting directory or an ancestor by a relative path records no repository.
---

### [LOW] A .. under a ~ or $ anchor is clamped at the anchor

**Location:** `internal/burlerengine/cluster.go` `resolveShellPath`.
**Issue:** The anchored branch cleans the rest as `path.Clean("/"+rest)`, so `$S/../target` resolves to `$S/target` and `~/..` to `~`.
After `git init "$S"`, `git -C "$S/../target" commit -m x` therefore lies inside the recorded `$S` and is excused, though the real directory is outside it.
The decision record says lexical resolution errs toward flagging and an unresolvable directory never matches a recorded repository; the clamp errs the other way.
The existing row for `~/r/../..` only covers a deeper root and does not catch this.
Reaching the target needs the anchor variable to hold a path beside it, so the exposure is small.

### [LOW] git init ../name records the starting directory

**Location:** `internal/burlerengine/cluster.go` `record`.
**Issue:** The starting-directory guard rejects only paths made entirely of `.` and `..` elements.
A fork that knows its own directory's basename can run `git init ../<basename>`, which is recorded as the relative root `../<basename>`, and a later `git -C ../<basename> commit` is excused.
The decision record names a relative path to the starting directory or an ancestor as never recorded.
Exploiting it requires knowing the target directory name, and the sibling case `git init ../x` is legitimate, so a fix that refuses any leading `..` errs toward flagging, as the record asks.

### [LOW] Added behaviours without a row

**Location:** `internal/burlerengine/cluster_test.go` `TestFirstMutatingGitCommandScopesToForkRepositories`.
**Issue:** Every acceptance criterion has a row, and no new row duplicates an existing one.
Four added behaviours have no row: the `--` terminator in `firstOperand`, the valued options in `gitInitValuedOptions` and `gitWorktreeAddValuedOptions` beyond `git init -b`, a trailing slash on a created path, and an anchored `cd` restoring a known directory after a bare `cd`.
A regression in any would pass the suite and fail a round that the record says should pass.

### [NIT] doc.go omits the git init bound

**Location:** `internal/burlerengine/doc.go:305`.
**Issue:** The paragraph says `git push`, `--git-dir`, `--work-tree` and an unresolvable directory never pass, but the record's fourth bound, that `git init` of the starting directory or an ancestor is never recorded, is described only in the code comment.

## Rejected

- Environment and config redirection (`GIT_DIR`, `GIT_WORK_TREE`, `-c core.worktree=`) from lens code-correctness: the decision record's bounds list only `--git-dir` and `--work-tree`, and these routes were never classified by the audit before the change, so they are an unlicensed widening of scope.
- Relative recorded roots persisting across commands while a real Bash session's cwd may change between them: the record decides that each command starts from the unknown starting directory, so this is the answer key's choice, not a diff defect.
- `git init ~` or `git init $HOME` recording a broad anchored root: the same class as the record's open risk about reinitializing the target by an absolute path.
- Lens code-affected-set and code-pattern-fit reported nothing, and I found no stale reference to `commandRunsMutatingGit` or `gitSubcommand`, no `PATTERN.md` change owed, and an unchanged error message and way forward.
