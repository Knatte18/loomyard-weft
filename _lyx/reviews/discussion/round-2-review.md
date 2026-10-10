---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:167
    summary: D12's lyxcwd import of gitrepo creates a test import cycle, because gitrepo's in-package gogit_test.go imports gitkit, which reaches lyxcwd through configengine and logger
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:194
    summary: The PATTERN-cwd-resolution constraint says D12 leaves the boundary unchanged, but lyxcwd's test-pinned import cap must widen to gitrepo, and the acyclicity rationale behind that cap then binds gitrepo
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:168
    summary: D12's walk-up skips an unusable gitfile and accepts any .git directory, which is the opposite of what rev-parse --show-toplevel does, so the claim that it matches git is false and a stale worktree resolves to an outer repository
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md:125
    summary: D8 moves only the watchdog recognizer into tmuxkit, but reedcli's /proc probes (pid list, argv, cwd), which D8's leftover scan also needs, stay behind, so PATTERN-testkit's single copy is not reached
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:245
    summary: The census-shows-the-drop criterion does not say whether the drop is read from the per-subcommand total or the code-side column, and D2's overlap bias makes the before-state's code column undercount
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:201
    summary: The test-budget constraint enumerates packages whose rows rise, and the list already omits loomcli, whose resume test D6 splits into two files
---

### [BLOCKING] D12's `lyxcwd` → `gitrepo` import makes `gitrepo`'s in-package tests an import cycle

**Location:** `_lyx/discussion/decision-record.md:167` (D12, first bullet), with the premise at `_lyx/discussion/support-log.md:356`.

**Issue:** D12 exports the `.git` entry reader from `gitrepo` and has `lyxcwd` import it.
The support log justifies this by saying `gitrepo`'s dependencies (`proc`, `gitexec`, `lock`) reach no `lyxcwd`.
That holds for production imports only.
`internal/gitrepo/gogit_test.go` is `package gitrepo` (an in-package test, `//go:build integration`) and imports `internal/gitkit` (27 `gitkit.` uses).
`go list -deps ./internal/gitkit` shows `gitkit → configengine → logger → lyxcwd`.
Once `lyxcwd` imports `gitrepo`, compiling `gitrepo`'s integration test package imports `gitrepo` through `gitkit`, which Go rejects as `import cycle not allowed in test`.
The decision is unbuildable as written, and the premise it rests on is false.

**Fix:** Settle where the reader lives so no cycle forms.
Either place the reader where neither `gitkit`'s closure nor `gitrepo`'s in-package tests can loop back (the support log's rejected "stdlib-only reader package" alternative, reconsidered with this evidence), or make D12 move `gogit_test.go`'s `gitkit` uses into an external test package, and state which in the Decision.
Correct the support-log premise either way.

### [MEDIUM] The `PATTERN-cwd-resolution` constraint understates what D12 changes

**Location:** `_lyx/discussion/decision-record.md:194`.

**Issue:** The constraint says D12 "changes its implementation, not its boundary" and updates the background's "imports gitexec only" line only "if the import goes".
`internal/lyxcwd/leaf_enforcement_test.go` pins `lyxcwd`'s import cap to `gitexec` alone through `scankit.AssertImportAllowlist`, and its header gives the reason: it keeps `fabricengine → logger → lyxcwd` acyclic.
D12 adds an import whether or not `gitexec` goes, so the pinned cap must widen in the same commit.
The widened cap brings go-git into `logger`'s closure and binds `gitrepo` never to import `logger` or anything that reaches `lyxcwd`.
Neither consequence is recorded, so the plan writer sees "boundary unchanged" and meets a failing enforcement test.

**Fix:** State that `lyxcwd`'s import allowlist and the background line both admit the reader's package, and record the new standing rule on that package's imports as part of the constraint, consistent with whatever F1 settles.

### [MEDIUM] D12's walk-up diverges from `--show-toplevel` on unusable `.git` entries

**Location:** `_lyx/discussion/decision-record.md:168`, and the criterion at line 241.

**Issue:** D12 walks up "to the nearest usable `.git` entry (a directory, or a `gitdir:` file whose target exists), as `--show-toplevel` does".
I checked git 2.53 in the scratchpad:
- A `.git` file whose `gitdir:` target is missing, inside an outer repository: `git rev-parse --show-toplevel` fails with `fatal: not a git repository: <target>` (exit 128); it does not walk on.
- An empty `.git` directory inside an outer repository: git resolves the outer repository; it does not accept the directory.

D12 does the opposite in both cases.
It skips the broken gitfile and resolves whatever repository encloses it, and it accepts any `.git` directory.
A pruned or moved worktree whose hub sits inside another repository then resolves `HubPath` to the parent of that outer repository instead of being refused.
The acceptance criterion promises "the same `Location` as today" but names neither case, so nothing pins the divergence.

**Fix:** Define "usable" as git does: a gitfile ends the walk, refused with `ErrNotAGitRepo` when its target is missing, and a `.git` directory counts only when it is a repository.
Add both cases to the `lyxcwd.Resolve` criterion.

### [MEDIUM] D8 leaves the `/proc` probes duplicated

**Location:** `_lyx/discussion/decision-record.md:125` (D8), Scope at line 32.

**Issue:** D8 moves `linuxIsWatchdogDaemon` from `internal/reedcli/smoke_proctree_test.go` into `tmuxkit` "so the kit holds the only copy".
That recognizer calls the file's `/proc` probes (`linuxPids`, `linuxProcArgv`, `linuxProcCwd`).
D8's own leftover scan needs the same probes: list pids and read each one's cwd, exe and argv.
Moving only the recognizer leaves two copies of `/proc` probing, one in `reedcli`'s tests and one in `tmuxkit`, which `PATTERN-testkit` forbids.

**Fix:** Have D8 and Scope move the `/proc` probes the leftover scan and the recognizer share into `tmuxkit`, with `reedcli`'s smoke test calling them from there.

### [LOW] The census-drop criterion does not name its column

**Location:** `_lyx/discussion/decision-record.md:245`, against D2's bound at line 73.

**Issue:** D2 states that the fixture/code split is approximate wherever hub builds overlap other tests.
In the before-state, every hub-taking test builds a hub, so overlap is the norm in hub-heavy packages and the code-side column undercounts.
After D11, hub builds become rare, so the code-side column grows more accurate.
If D12's drop is read from the code-side `rev-parse` column, that bias hides part of the drop.
The criterion says only that "the census shows the drop".

**Fix:** Choose the measure: either the per-subcommand total, which the bias does not affect, or a split taken from a census-only run with `-parallel 1`, and write the choice into the criterion and D3.

### [LOW] The test-budget constraint pins an incomplete package list

**Location:** `_lyx/discussion/decision-record.md:201`.

**Issue:** The constraint lists the packages whose `test-budget.yaml` rows rise.
D6 splits `TestLoomResumeReconcilesTheHubConfigBeforeArming` into a tier-2 half and a `tmux` file, which can add a top-level test in `loomcli`.
Moving reed-starting halves elsewhere can do the same in `reedcli` and `orchcli`.
None of these packages is on the list.
An enumerated list like this goes stale when the plan adds a package.

**Fix:** Replace the list with the rule: a row rises in the same commit as the tests that push its package past it, as CLAUDE.md's test-budget rule says.

## Focus verdicts with no finding

- D12, anchored subpath: walking up from `<worktree>/backend` reaches the worktree's `.git`, and the gate stays unchanged, so the anchor directory resolves and the gate still applies.
  Today's not-a-repo path returns the bare `ErrNotAGitRepo`, which matches "today's message", and the criterion now names a subpath anchor.
  The remaining D12 defects are F1 to F3.
- D2: `hubforge` has its own git helper (`hub.go`), and the process-wide marker reaches `gitexec` children because `gitexec` inherits `os.Environ()`.
  Hub-build git is attributed to fixtures, and the overlap bound is stated; F5 covers how the comparison reads that bias.
- D7: a build with non-empty ldflags never returns the prebuilt binary, and the criterion covers `BuildWithLDFlags` set or unset.
- D11: (a) the scan's banned tokens now include both constructors; (b) poisoning stops later `SharedHub` and `CopyHub` calls, and the race window is bounded; (c) the write definition covers in-process verbs and every spawned git.
- D13: the allowlist matches today's selector sites (`shuttleengine`, `loomcli`, `orchcli`, `reedcli`, `reedengine`, `shuttlefake`), and a selector match catches method values as well as calls.
  Scope now names `claudeengine`, the per-package runners and the recognizer's source; F4 covers the probes the recognizer depends on.
