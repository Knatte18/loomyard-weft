---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:163
    summary: D12 retires every fabricengine rev-parse but the disambiguation sites, which removes --git-path hooks and reintroduces the core.hooksPath bug hook.go records as fixed
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:113
    summary: D8's claim that tmuxkit holds the only copy of the watchdog recognizer and /proc probes is false; loomcli and orchcli keep their own, so PATTERN-testkit stays violated
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:159
    summary: D12's repository test accepts a git dir holding HEAD and a commondir file even when the commondir target is gone, where git refuses it, so the walk-up departs from --show-toplevel
---

### [BLOCKING] D12 removes the `--git-path hooks` spawn that honours `core.hooksPath`

**Location:** `_lyx/discussion/decision-record.md:163`, with the acceptance criterion at line 240.

**Issue:** D12 says `fabricengine`'s `rev-parse` spawns move onto the in-process geometry and ref reads, and that a `rev-parse` survives only where the answer is git's own disambiguation (`--disambiguate`, `<commit>:<path>` existence).
The acceptance criterion at line 240 repeats that only the disambiguation sites survive.
`internal/fabricengine/hook.go:64-72` runs `rev-parse --git-path hooks`, and its comment records why: composing `<git-common-dir>/hooks` ignores `core.hooksPath`, so on a repository that sets it fabric wrote the hook where git no longer looks, reported success, and the drift warning never fired.
That answer depends on config, not on geometry, so neither a geometry join nor a ref read reproduces it.
As written, D12 makes the plan writer replace the spawn with a join and reintroduce that defect.
D12's premise that every `rev-parse` outside disambiguation is a geometry or ref lookup is false for this site.

**Fix:** In D12, widen the survival rule from "git's own disambiguation" to "an answer git derives from config or revision syntax", or name `--git-path hooks` as a surviving spawn on the `core.hooksPath` ground.
Amend the acceptance criterion at line 240 to match.

### [BLOCKING] D8 leaves duplicate `/proc` probes and watchdog recognizers outside `tmuxkit`

**Location:** `_lyx/discussion/decision-record.md:113`, and Scope at lines 19 and 21.

**Issue:** D8 moves the watchdog recognizer and the shared `/proc` probes from `reedcli`'s `smoke_proctree_test.go` into `tmuxkit` and concludes "so the kit holds the only copy of each".
That premise is false.
`internal/loomcli/smoke_helpers_test.go:274-313` has its own watchdog recognizer (`findWatchdogPIDs`, matching `reed watchdog` plus `--hub-path`) and its own `/proc` pid listing, argv read and cwd read (`findDriverPIDs`).
`internal/orchcli/smoke_live_test.go:441-458` (`killTreeProcs`) lists pids and reads argv and cwd to kill whatever references a directory, which is D8's leftover scan in miniature.
Once `tmuxkit` exports these probes, each package-local copy is duplicated test support, which `PATTERN-testkit` forbids, and D10 already moves `lyx` runners onto `lyxbin` on that same ground.
This is not a demand for a site list: D8 states a sourcing rule whose conclusion does not hold, and the plan writer, who never reads `support-log.md`, would take "the only copy" as settled.

**Fix:** Replace D8's `reedcli`-only move with a rule: every test-side watchdog recognizer and `/proc` pid, argv or cwd probe moves onto `tmuxkit`'s exported probes, and no test package keeps its own.
Change Scope lines 19 and 21 to state that rule instead of naming `reedcli` alone.

### [MEDIUM] D12's repository test ignores whether `commondir` resolves

**Location:** `_lyx/discussion/decision-record.md:159-160`.

**Issue:** D12 calls a git dir a repository when "it holds `HEAD` and either a `commondir` file or `objects` and `refs` directories", and applies the same test to a gitfile's target.
Git checks `objects` and `refs` in the directory `commondir` names, not only that the file exists.
Probed with git 2.53.0: a gitfile pointing at a dir holding `HEAD` and a `commondir` file naming a missing path makes `git rev-parse --show-toplevel` fail with `fatal: not a git repository` and exit 128.
Under D12's test that target counts as a repository, so `lyxcwd` resolves it.
That case is a linked worktree whose primary repository was moved or deleted, the same pruned-worktree case support-log line 48 gives as the reason to refuse rather than walk on.
D12's claim that the walk-up behaves "as `--show-toplevel` does" therefore fails here, and no acceptance criterion pins this case.
The two cases the focus directive names do match git: a gitfile naming a missing git dir exits 128, and an empty `.git` directory resolves to the outer repository, and line 236 pins both.

**Fix:** Change D12's repository test to: `HEAD` present, and `objects` and `refs` present in the common dir, which is the `commondir` target when that file exists and the dir itself otherwise.
Add a row to the `lyxcwd.Resolve` acceptance criterion for a gitfile whose `commondir` target is missing, refused with `ErrNotAGitRepo`.

## Verdicts on the focus directive

- D12 import graph: no cycle.
  `dotgit` imports the standard library alone, so nothing it compiles can reach `lyxcwd`.
  `go list -deps ./internal/gitkit` includes `lyxcwd`, so `gitrepo`'s in-package tests that import `gitkit` compile `lyxcwd -> dotgit`, and that path is acyclic.
  `lyxcwd`'s own tests that import `gitkit` are already in `lyxcwd_test` (`testmain_test.go`), and its in-package `leaf_enforcement_test.go` imports only `scankit`, whose deps do not include `lyxcwd`.
  Not a defect.
- D12 walk-up: matches git for the two named cases, and line 236 pins both.
  It diverges for a missing `commondir` target, filed as F3.
- `PATTERN-cwd-resolution` constraint, line 186: it states the cap change, the background line, and the `leaf_enforcement_test.go` allowlist, and `dotgit`'s own stdlib-only rule sits under `PATTERN-leaf-packages` at line 188.
  `docs/shared-libs/lyxcwd.md` lines 6 and 121 also state the old cap, but listing every doc that restates it is cross-reference enumeration, which the rubric excludes.
  Not a finding.
- D8 and Scope `/proc` probes: not one copy, filed as F2.
- Census column: D3 line 67 and the acceptance criterion at line 240 both read the `rev-parse` drop from the per-subcommand total, which D2's overlap bias does not move.
  Consistent, not a defect.
- Test-budget constraint, line 195: it states a rule, a row rising in the same commit as the tests that push the package past it, and lists no packages, so no package D6 or D8 splits can be missing.
  Not a defect.
