---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gitexec/transport.go#namesMissingLocalPath
    summary: "A remote name that is not configured is classed as a transport failure, since git prints 'origin' does not appear to be a git repository and the code stats <dir>/origin, finds nothing and returns true"
    fix: "In namesMissingLocalPath, return false when the matched string holds no path separator and no file:// prefix, so a bare remote name such as origin is never stat-ed as a relative path. State in the IsTransportFailure doc comment that an unconfigured remote is a refusal. In transport_test.go add a hand-built row with Args push origin and Stderr \"fatal: 'origin' does not appear to be a git repository\" wanting false. In TestIsTransportFailure_RealGit, run ls-remote against a remote name the repository does not configure and assert IsTransportFailure is false."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:18
    summary: The change description lists only the push classification and ReconcileOrigin as following callers, and its blast radius names only fabric, while shedtransient.Class outside fabric, outcomeClassOf, RetryOriginReadInTransit, withTransportCause, the owed records push and awaitOrigin also change behaviour
    fix: "In _lym/landing/summary.md, extend the caller list under The change to name every IsTransportFailure caller whose behaviour flips: shedtransient.Class now marks the failure as a transient git transport wait, outcomeClassOf reports OutcomeTransportFailure, RetryOriginReadInTransit and retryTransportPush and awaitOrigin back off and retry, withTransportCause joins the probe error, and the owed records push records a transport failure. Change the Merge danger blast radius to name fabric and shed."
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: internal/gitexec/transport.go:88
    summary: os.Lstat counts a dangling symlink as existing, so an origin reached through a symlink into an unmounted share reads as a refusal, against the doc claim that an unmounted origin directory reads as unreachable
    fix: "In namesMissingLocalPath replace os.Lstat with os.Stat, so a link whose target is gone reads as missing, and add a transport_test.go row whose path is a symlink to a missing target, wanting true."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gitexec/doc.go:10
    summary: The package doc and the IsTransportFailure doc say an unmounted origin directory reads as unreachable, but an origin URL naming the mount point itself still exists as an empty directory after unmount and is classed as a refusal
    fix: "In internal/gitexec/doc.go and the IsTransportFailure doc comment in internal/gitexec/transport.go, say that a repository path that is gone, such as a removed directory or one below an unmounted mount point, reads as unreachable, and add a Bound line that a mount point left as an empty directory reads as a refusal."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gitexec/transport.go:74
    summary: "The [^']+ group stops at an apostrophe, so for a path such as /x/it's.git the match is the relative s.git, which is joined under dir and usually missing, so a non-repository directory whose name holds an apostrophe reads as unreachable"
    fix: "In missingRepositoryPattern, match the quoted path greedily within one line, as '([^\\n]+)' does not appear to be a git repository and repository '([^\\n]+)' does not exist, and add a transport_test.go row whose existing non-repository directory name holds an apostrophe, wanting false."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gitexec/transport_test.go:64
    summary: The file:// row and the relative-path row both use a missing path, so they still pass with the TrimPrefix or the join against Dir removed
    fix: "In TestIsTransportFailure add two rows wanting false: an ls-remote whose stderr names 'file://' plus the existing directory, and a fetch whose Dir is the parent of the existing directory and whose stderr names the existing directory's base name as a relative path."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gitexec/transport.go:87
    summary: A relative remote URL is resolved against the directory git ran in, but git resolves it against the worktree top, so a run from a subdirectory stats the wrong path
    fix: "In the IsTransportFailure doc comment add a Bound line that a relative remote path is read against the directory git ran in, which matches git only when that directory is the worktree top, as every lym call site runs it."
---

### [MEDIUM] An unconfigured remote name reads as unreachable

**Location:** `internal/gitexec/transport.go#namesMissingLocalPath`

**Issue:** Real git 2.53.0 prints `fatal: 'origin' does not appear to be a git repository` for push, fetch and ls-remote against a remote name it has no configuration for.
The new pattern matches that line, treats `origin` as a relative path, joins it under `GitError.Dir`, finds nothing and returns true.
A repository with no origin remote is a supported state (`internal/fabricengine/recordslayout.go:41`), and fabric runs remote commands against the remote name `origin`.
A misconfiguration that retrying never fixes is then retried, recorded as an owed `transport` push or reported as an unreachable origin, instead of stopping.
No test pins either verdict.

### [MEDIUM] The change description names fewer affected callers than the diff changes

**Location:** `_lym/landing/summary.md:18`

**Issue:** The summary says every caller follows and names only the push classification and `ReconcileOrigin`.
`IsTransportFailure` also drives `shedtransient.Class` (`internal/shedtransient/shedtransient.go:34`), which lies outside fabric and now marks the failure as a transient wait, and `outcomeClassOf`, `RetryOriginReadInTransit`, `retryTransportPush`, `awaitOrigin`, `withTransportCause` and the owed records push in `pushanchored.go:459`.
The `Merge danger` blast radius of `fabric` therefore names less than the diff can break.
The `two-way` door holds: no record format, config meaning or outward push changes.

### [LOW] Lstat counts a dangling symlink as an existing path

**Location:** `internal/gitexec/transport.go:88`

**Issue:** An origin reached through a symlink into an unmounted share leaves the link in place, so `os.Lstat` succeeds and the failure is a refusal, while the doc comment says an unmounted origin directory reads as unreachable.

### [LOW] The unmounted-directory claim overstates what the code classifies

**Location:** `internal/gitexec/doc.go:10`

**Issue:** An origin URL that names the mount point itself leaves an empty directory once the share is unmounted.
That path exists, so the code classes the failure as a refusal, while `doc.go` and the `IsTransportFailure` doc comment say an unmounted origin directory reads as unreachable.

### [LOW] An apostrophe in the path breaks the path capture

**Location:** `internal/gitexec/transport.go:74`

**Issue:** Real git prints `'/…/it's.git' does not appear to be a git repository`.
The `[^']+` group matches only `s.git`, which is then stat-ed under `dir`, so the verdict rests on an unrelated path.

### [LOW] Two new rows cannot fail for the behaviour they name

**Location:** `internal/gitexec/transport_test.go:64`

**Issue:** The `file://` row passes without the `TrimPrefix`, because the joined path is missing too.
The relative-path row passes without the join against `Dir`, because the bare name is missing from the test process's directory too.

### [LOW] A relative remote path is resolved against git's cwd, not the worktree top

**Location:** `internal/gitexec/transport.go:87`

**Issue:** Run from a subdirectory, git resolved the relative URL `gone.git` against the worktree top and succeeded, while the code would join it under the subdirectory.
Every lym call site runs git at a worktree root and none passes `-C`, so the gap is unstated rather than reached.

## Rejected

- The scope lens found no finding; every goal point of the board entry is built, both places named by the entry now report unreachable, and the `Root-Cause:` trailer names the defect the diff removes.
- A stat failing with EACCES or EIO reads as a refusal: the contract states nothing about it, and the lens raised no finding.
- The `~/x` path joined under `dir`: git prints it unexpanded, and only a `~` directory inside the worktree would misclassify it.

## Focus departures

- "do the matched stderr lines hold under the environment gitexec runs git in (locale)": this machine has no git message catalogue installed, so a translated stderr could not be produced with real git.
  `childEnv` sets no `LC_ALL` or `LANG`, so the new pattern depends on the operator's locale exactly as the existing `transportPhrases` do; no finding was raised for a gap the diff did not open.
  The real-git test pins push, fetch, ls-remote and clone; pull is pinned only by its shared fetch stderr.
