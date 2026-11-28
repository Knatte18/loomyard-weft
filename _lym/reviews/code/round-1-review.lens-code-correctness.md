```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each new function and branch in the diff on concrete inputs, using real git 2.53.0 where it matters.

**Per function or branch: the case run, and whether the result is correct**

- **`missingRepositoryPattern` / `namesMissingLocalPath`, absolute missing path** (`'/x/removed.git' does not appear…`): stat finds nothing, so the error counts as transport. Correct.
- **`file://` URL**: git prints the decoded path without `file://`, and `TrimPrefix` also covers the hand-built row. Correct.
- **Percent-encoded `file://` URL**: git prints `a b.git` already decoded. Correct.
- **`clone` of a missing plain path**: git prints `repository '&lt;p&gt;' does not exist`, which group 2 matches. Correct.
- **`clone` of a missing `file://` URL**: git prints the `does not appear` form, which group 1 matches. Correct.
- **Push to an existing directory with no repository**: stat succeeds, so the error stays a refusal. Correct.
- **Unconfigured or mistyped remote name** (`git push origin` with no `origin` remote, or `git push orgin`): git prints `'origin' does not appear to be a git repository`. The path is relative, so the code joins it as `Dir/origin`, which is missing, and the error counts as **transport. Wrong.**
- **Dangling symlink origin** (`link.git` pointing at `/nonexistent-mount/x.git`): git names `link.git`. `os.Lstat` finds the link, so the error stays a **refusal. Wrong** for the unmounted-share case the doc comment names.
- **Relative URL with a `-C` global option** (Dir = `$S`, args `-C r ls-remote origin`, URL `../gone.git`): git resolves the URL against `$S/r`, while the code joins it to `$S`. **Wrong base.** Both paths were missing in this probe, so the result happened to agree.
- **Relative URL run from a subdirectory**: git resolves against the worktree top (the push from `r/sub` with `../bare` succeeded), while the code joins it to Dir. **Wrong base** whenever Dir is a subdirectory. lym normally runs git at a worktree root, so the effect is small.
- **Empty `GitError.Dir`**: `filepath.Join("", p)` is `p`, which is stat-ed against lym's working directory, the same directory git ran in. Correct.
- **`~/x` URL**: the code joins it as `Dir/~/x`, which is always missing, so the error always counts as transport. git would read `$HOME/x`, so an existing non-repository directory at `$HOME/x` is misclassified. **Wrong, edge case.**
- **Path holding an apostrophe** (`…/it's.git`): `[^']+` cannot cross the apostrophe, so the match starts there and captures `s.git`. That is joined to Dir and is missing, so the error counts as transport even when `it's.git` exists and holds no repository. **Wrong, edge case.**
- **Non-English locale**: git translates the message, the pattern does not match, and the classification falls back to `other`. This fails safe.
- **`IsTransportFailure`'s HTTP 4xx refusal and network phrases** run before the new check and keep their order. Correct.
- **`IsTransportFailure` when the error has a Timeout** returns before the new check. Correct.

**Findings**

1. **MEDIUM, design**, `internal/gitexec/transport.go:79-91` (`namesMissingLocalPath`): an unconfigured or mistyped remote name counts as a transport failure.
   - git quotes the remote name, the code stats `Dir/&lt;name&gt;`, finds nothing and returns true.
   - So a store with no `origin` remote, or any command whose remote name is wrong, becomes an owed `transport` push retried quietly, or an `Unreachable` reconcile, when it should stop as a refusal.
   - Evidence: real git output `fatal: 'origin' does not appear to be a git repository` and `fatal: 'orgin' …`, from my probe in `.scratch/exp1`. Several fabric doc comments describe a repository with no origin remote, for example `internal/fabricengine/weftbranch.go:21`.
   - Fix: only stat a captured string that is a path or a `file://` URL: absolute, or starting with `./`, `../` or `file://`. A bare word is a remote name, and should count as transport only when that remote is configured with a URL.
2. **LOW, design**, `internal/gitexec/transport.go:88`: `os.Lstat` classes a dangling symlink origin as an existing path, so a symlink to an unmounted share stays a refusal.
   - Evidence: my probe in `.scratch/exp2`, where git named `…/link.git` and the link target was missing.
   - Fix: use `os.Stat`, which follows the link the way git does.
3. **LOW, design**, `internal/gitexec/transport.go:85-87`: a relative path is resolved against `GitError.Dir`, but git resolves it against the `-C` directory, or the worktree top when run from a subdirectory.
   - This contradicts the doc comment, which says "the directory git ran in".
   - Evidence: my probe, `-C r ls-remote origin` with `../gone.git`, and the push from `r/sub` that succeeded with `../bare`.
   - Fix: apply the args' `-C` operands to Dir. Alternatively, narrow the doc comment and accept the mismatch, since lym runs no `-C` remote commands in production (grep showed none).
4. **LOW, design**, `internal/gitexec/transport.go:74`: an apostrophe inside the path makes the regex capture only the tail after it, and a `~/` path is joined to Dir instead of `$HOME`.
   - Either case can class an existing directory holding no repository as transport.
   - Evidence: real git output `'…/it's.git' does not appear…` and `'~/definitely-missing-xyz.git' …`, from my probe in `.scratch/exp1`.
   - Fix: anchor the match on the full line, `fatal: '(.*)' does not appear…$` with a greedy capture, and expand a leading `~/`.

Tool calls:
1. Ran git against a dangling-symlink origin and a relative URL under `-C` in `.scratch/exp2`. The context had no probe of either case.
