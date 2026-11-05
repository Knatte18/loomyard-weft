---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/home.go:78
    origin: lens:code-tests
    summary: Symlink resolution of the git dir and common dir in readCheckout has no test row that fails when it is dropped.
    fix: In internal/shuttleengine/codexengine/prepare_test.go add a TestPrepare_Launch row (for example GitDirsThroughLink) whose fixture reaches the git dir and the common dir through a symlink, so the worktree's `.git` gitfile names the `gitdir:` through the link and its `commondir` resolves through it too. Build the link with os.Symlink under repo.base in fixture_test.go, and have the row expect the resolved paths that wantGitDirs returns. Removing either EvalSymlinks call in readCheckout then fails the DeepEqual on the launch.
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/config.go:137
    origin: lens:code-tests
    summary: A leading ~/ in writable_roots, the form the shipped template default uses, is expanded by configuredWritableRoots with no test pinning it.
    fix: Add a test in internal/shuttleengine/codexengine/prepare_test.go that sets HOME with t.Setenv to a directory under repo.base, creates a cache folder there, names it as ~/cache in writeWritableRoots, and expects the resolved folder among the launch's writable roots. It cannot call t.Parallel because it sets an environment variable, so a comment above it names the environment as the reason, as PATTERN-test-speed requires. Keep it a separate test function, since a subtest of the parallel table cannot use t.Setenv.
---

### [LOW] Git dir symlink resolution is unpinned by any test

**Location:** `internal/shuttleengine/codexengine/home.go:78`
**Issue:** `readCheckout` now resolves `geometry.GitDir` and `geometry.CommonDir` with `EvalSymlinks`, and `doc.go` and the summary claim both paths are symlink-resolved because Codex keys a path by its resolved target.
`newRepoFixture` resolves its own temp base, and no row reaches either git dir through a link, so removing either call changes no expectation.
Only the records checkout has a through-a-link row (`RecordsToldThroughLink`).
This is a behavior the diff adds with no coverage.

### [LOW] The `~/` form of `writable_roots` is unpinned by any test

**Location:** `internal/shuttleengine/codexengine/config.go:137`
**Issue:** The template default `~/.cache/go-build` reaches the launch only through `expandHomePath`'s `~/` branch inside `configuredWritableRoots`.
Every test row names an absolute path, and `TestLoadConfig_Template` pins only the raw template text.
A change that dropped the expansion would make `EvalSymlinks` see `~/.cache/go-build`, find it missing and skip it, and every test would still pass.

## Rejected

- Focus point 1, whether a `writable_roots` default meets the entry's Go cache and temp half: the entry says "such as `GOCACHE` under the lym temp root", so the temp-root location is an example and the requirement is a writable Go cache location. The key makes `~/.cache/go-build` writable, which answers the read-only-cache refusal the entry names. The entry's temp half needs no change: the entry's own account has the writer's `/tmp` cache working under the same sandbox, so a temp location is already writable. The new key, its `CategoryGuard` and the template line are licensed by the entry's "go cache and temp paths need a writable location too", and the summary states the choice as a design decision. Not a defect.
- Focus point 2, `EvalSymlinks` failing the launch on a missing git dir: `dotgit.FindRoot` returns success only after `IsRepository` has seen `HEAD` in the git dir and `objects` and `refs` in the common dir, and `Read` requires the `gitdir:` and `commondir` targets to exist, so no layout it accepts reaches the failure where the old code launched. Order and dedupe in `gitDirs` hold for a plain clone (one dir) and a linked worktree (git dir, then common dir). Not a defect.
- Focus point 3, widening the sandbox to the common git dir for every session: the entry asks for exactly this ("the worktree's git directory, and the common git directory it links to"). The order of roots, the Warn path for an unresolvable records checkout and the append of the configured roots after the git dirs match the `readyHome` doc comment. Not a defect.
- Focus point 4, `configuredWritableRoots` on a non-directory path, a dangling symlink and a permission error: a dangling symlink returns `ErrNotExist` and is skipped, a permission error refuses the launch naming the key, and `expandHomePath` is the one gate that refuses an empty or relative value by its key. A configured regular file is added as a root and a path under a file refuses the launch (lens code-correctness finding 1, LOW): both are an operator's misconfiguration of their own config, the refusal names the key, and a file as a writable root widens nothing beyond that file. Rejected as not worth a fix.
- Focus point 5, `CategoryGuard`: `LocalDenied` lists every key classified in a category other than `CategoryNone`, so a local override of `writable_roots` is refused, the commit message's claim holds, and `configreg_test.go` and the shuttle row of `docs/overview.md` follow from it. Not a defect.
- Lens code-correctness finding 2 (NIT, the `EvalSymlinks` refusal in `configuredWritableRoots` has no `way forward:` clause): `PATTERN-refusal-way-forward` binds only webster, shed and loom, and the sibling `readCheckout` refusals added by the same diff also carry none. Not a rule violation.
- Lens code-tests finding 3 (NIT, no row for an empty `writable_roots` entry): the relative-value row reaches the same `expandHomePath` branch, and the test-economy rule makes a further row for it redundant.
- Whether the `~/.cache/go-build` template default breaks `PATTERN-no-hard-coded-choice`: it is a config value, not a code path branching on a language, and a hub without the folder adds nothing. Not a defect.
- The commit messages carry no `Pattern-Departure:` trailer, the directive declares no PATTERN changes, and `_lym/landing/summary.md` makes no claim the diff does not bear out. Its closing note says no Codex session ran against the new launch line, and nothing here contradicts that.
