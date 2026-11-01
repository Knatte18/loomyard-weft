---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: tools/boltrename/main.go:50
    summary: The liveness check compares symlink-resolved /proc cwds with an unresolved hub path, so a hub entered through a symlink is missed by the cwd test and the check does not fail closed.
    fix: In main, after filepath.Abs, resolve hubPath with filepath.EvalSymlinks (failing on error) and pass the resolved path to liveProcesses and inspectHub; keep the Abs path as a second string the command-line match also tests. Add a TestLiveProcesses row whose Cwd is the resolved path.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: tools/boltrename/rename_test.go:93
    summary: Most of the Design's stop paths and the self/ancestor skip have no test row; only four stops and the live-process filter are pinned.
    fix: Add rows to the TestInspectHub_StopsOnWhatItDoesNotExpect table for a stray file in the hub, no prime with a weft, a wrong checkout .git link, and a pre-existing new admin dir; add a TestLiveProcesses row with PID os.Getpid() that must not be listed. Add rows to the existing tests, not new test functions.
    origin: lens:code-tests
---

### [LOW] Liveness check can miss a hub reached through a symlink

**Location:** tools/boltrename/main.go:50
**Issue:** `filepath.Abs` does not resolve symlinks, while `/proc/<pid>/cwd` does. When `-hub` names the hub through a symlink, `within(hubPath, p.Cwd)` never matches a process working inside the hub, leaving only the exe and command-line tests. The Design makes refusing while any run is alive the program's core safety property, so an ambiguous path should fail closed.

### [LOW] Stop and liveness paths partly untested

**Location:** tools/boltrename/rename_test.go:93
**Issue:** The stop branches for a stray hub file, a missing prime/weft, a wrong checkout `.git` link and a pre-existing new admin directory (`main.go:120,138-150`) have no row, and nothing exercises the own-PID/ancestor skip at `main.go:76`. These are the Design's "stops and says what it found" behaviours. No test duplicates another's coverage.

## Rejected

- Lens code-correctness F1 (half-renamed hub after a mid-run failure names no progress): the Design excludes partial-failure handling, and the `os.Rename` error already names both paths.
- Lens code-correctness F2 (`oldToken` has no trailing boundary, so `_boards` or `_boardroom` would be rewritten): the Design rewrites every `_board` spelling; the hubs on Neve and OSL-1046 hold no such names, and the program is not meant to handle layouts beyond them.
- Lens code-tests, `main` refusal on a `ListProcesses` error: `main` is untested by design, and `liveProcesses` holds the logic. The lens called it an accepted gap.
- Lens code-tests NIT (the `ErrNotExist` and unchanged-file branches in `renameBolt`): thin branches in a throwaway program; no coverage required.
- Non-Go swap (calls.tsv, transcript.jsonl, research records): the Design says every remaining `_board` spelling in docs "and anywhere else" becomes `_bolt`, so the swap is licensed. The diffs I read touch only paths and prose, and no measurement column changed.
- Scope and constant: `lymcwd.BoltDirName` is the one value, `fabricengine.BoltDirName` aliases it, and `_board` remains only in `tools/boltrename` and in identifiers such as `custom_board`. No fallback, detection or migration case exists in lym code, and no refactor rides along. The geometry enforcement walks `internal` and `cmd` only, so `tools/boltrename`'s `_launchers` literal is not policed.
- Commit messages and `_lym/landing/summary.md` match the diff.
