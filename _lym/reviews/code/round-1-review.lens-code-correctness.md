```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes, one concrete input at a time.

**List**

- `missingRepositoryPattern` with the `does not appear to be a git repository` form, run on real git output for push, fetch and ls-remote. Correct: group 1 holds the path.
- `missingRepositoryPattern` with the clone form `repository '../gone.git' does not exist`. Correct: group 2 holds the path.
- `missingRepositoryPattern` with a path holding an apostrophe, `'/…/exp1/it's.git' does not appear…` (real git). Wrong: `[^']+` cannot cross the apostrophe, so the match is `'s.git'` and the path read is the relative `s.git`.
- `namesMissingLocalPath`, no match (for example `couldn't find remote ref`). Correct: returns false.
- `namesMissingLocalPath`, absolute missing path. Correct: true.
- `namesMissingLocalPath`, absolute path that exists. Correct: false.
- `namesMissingLocalPath`, `file://` URL with a percent-encoded path. Correct: git prints the decoded plain path with no `file://`, so the prefix strip does not apply and the path is read as given.
- `namesMissingLocalPath`, a remote name that is not configured. Real git prints `'origin' does not appear to be a git repository`. Wrong: `filepath.Join(dir, "origin")` does not exist, so the result is true and a misconfigured remote is classed as a transport failure.
- `namesMissingLocalPath`, relative URL run from a subdirectory of the worktree. git resolves the URL against the worktree top; my probe ran ls-remote from `sub` with the repository at the top's `gone.git`, and it succeeded. Wrong: the code joins the path to the subdirectory instead.
- `namesMissingLocalPath`, empty `Dir` with a relative path. Correct: `filepath.Join("", p)` resolves against the process cwd, which is also where git ran.
- `namesMissingLocalPath`, `~/x` path. git prints it unexpanded and the code joins it under `dir`. A missing path classifies true; a `dir/~` that happens to exist would misclassify. Negligible.
- `namesMissingLocalPath`, path that is a dangling symlink (for example to an unmounted share). Wrong against the doc comment: `os.Lstat` succeeds on the link itself, so the result is false and the origin is classed as a refusal.
- `namesMissingLocalPath`, `Lstat` failing with EACCES or EIO (for example a stale network mount). Returns false, so the failure is a refusal. The contract states nothing here and a hung mount is arguably transient; I raise no finding.
- `IsTransportFailure`, final branch. A transport phrase or a timeout still wins first and an HTTP 4xx refusal still returns false before it. Correct ordering.
- `IsTransportFailure`, local subcommand naming a missing path. Correct: false.
- Callers of the branch:
  - `judgeAgainstOrigin` and `lsRemotePair`: a missing path is retried under the remote-refs locks, then reports `Unreachable` with a nil error. Consistent with the summary.
  - Push classification at `pushclassified.go:647`: a missing path is recorded as a `transport` failure.
  - `retryTransportPush` and `awaitOrigin`: a missing path now sleeps through the push backoff before it is recorded. This is a new wait, but bounded by the backoff, so no finding.

**Findings**

1. MEDIUM, design. `internal/gitexec/transport.go:79-90`, `namesMissingLocalPath`.
   - A remote name that is not configured is classed as a transport failure.
   - With no `origin` remote, git prints `fatal: 'origin' does not appear to be a git repository`; the code stats `<dir>/origin`, finds nothing and returns true.
   - The result is misconfiguration retried and recorded as an owed `transport` push, or reported as unreachable by the reconcile, instead of a refusal. Production code runs remote commands against the remote name `origin` (`internal/fabricengine/recordsprobe.go:227`, `destroy.go:37`).
   - A repository with no origin remote is a supported state per the doc comment at `recordslayout.go:41`.
   - Evidence: real git 2.53.0 probe output, `== unconfigured remote name`.
2. LOW, design. `internal/gitexec/transport.go:85-87`.
   - A relative remote URL is resolved against git's cwd (`GitError.Dir`), but git resolves it against the worktree top.
   - Run from a subdirectory, the code can stat the wrong path in either direction. The probe: `env -C $W/sub git ls-remote origin` with URL `gone.git` succeeded against `$W/gone.git`.
   - A `-C <dir>` global option is ignored the same way.
   - No production site passes `-C` to git (grep found only `go build -C`), so the reachable case is a run from a subdirectory.
3. LOW, design. `internal/gitexec/transport.go:88`.
   - `os.Lstat` treats a dangling symlink as an existing path.
   - An origin reached through a symlink to an unmounted share therefore reads as a refusal, which contradicts the "removed or unmounted directory counts" contract at `transport.go:95` and `doc.go:9-10`.
   - `os.Stat` follows the link and would report it missing.
4. LOW, design. `internal/gitexec/transport.go:74`.
   - The quoted-path group `[^']+` cannot hold an apostrophe. For `'/…/it's.git'` the match is `'s.git'`, which is relative and gets joined to `dir`, so the stat runs on an unrelated path.
   - Usually that path is missing, so a directory holding no repository whose name has an apostrophe is misclassed as unreachable.
   - Evidence: real git probe output, `== apostrophe`.

Tool calls: one grep for production `"-C",` git calls and for the origin remote name, because the manifest had not shown whether any production site passes `-C` to git or names `origin`.
