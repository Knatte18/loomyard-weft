---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: cmd/lym/fslinkonly_test.go:46
    summary: A file that dot-imports os is skipped whole, so an unqualified Symlink call passes the fs-links scan although D4 says any name the file binds os to
    fix: >-
      In linkCallFailures in cmd/lym/fslinkonly_test.go, stop relying on osImportName alone for a dot import. Check file.Imports for a dot import of "os" (imp.Name.Name == "." with path "os") and report it as a failure, for example "dot import of os hides link calls; import os by name", outside internal/fslink. Add two fixture rows to TestFslinkOnly_FixtureTrees, a dot import with a bare Symlink call expecting 1 failure, and an `import _ "os"` file expecting 0. Leave osImportName unchanged because baretemp_test.go shares it.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: cmd/lym/interactivespawn_test.go:114
    summary: The allowlist is consulted for every walked file before any finding exists, so an entry for a file with no remaining finding never reports stale, and admission has no test row
    fix: >-
      In headlessSpawnFailures in cmd/lym/interactivespawn_test.go, remove the early `if allow.Allowed(rel) { return }` at line 114. Collect the file's findings in report as now, and call allow.Allowed(rel) once after ast.Inspect only when at least one finding was collected, dropping them when it returns true. Add two fixture rows beside the existing ones, built with a non-nil allowlist, for example scankit.NewAllowlist([]scankit.Entry{{Key: "p/a.go", Why: "fixture"}}). One row has a violating p/a.go and expects 0 failures. The other has a clean p/a.go and expects the allowlist's Stale() to list "p/a.go". This needs the fixture loop to take an optional allowlist field.
---

### [MEDIUM] Dot import of os hides link calls from the fs-links scan

**Location:** `cmd/lym/fslinkonly_test.go:46`, through `osImportName` at `cmd/lym/baretemp_test.go:40`.
**Issue:** `osImportName` returns `""` for a dot import of `os`, and `linkCallFailures` then returns early for that file.
`import . "os"` followed by `Symlink(...)` is a link call the scan never sees, and no fixture row records the gap.
Decision record D4 says the scan reports each call "through any name the file binds `os` to", and the focus asks how every binding is handled.
The aliased import is covered; the dot import is not.
Origin: kept from the code-affected-set lens at MEDIUM; the code-correctness and code-tests lenses raised the same gap at LOW.

### [MEDIUM] Spawn-scan allowlist never reports a stale entry and its admission is untested

**Location:** `cmd/lym/interactivespawn_test.go:114`.
**Issue:** `allow.Allowed(rel)` runs for every walked file, before any finding is computed, and it records the entry as used.
An entry keyed to a file whose findings have since been fixed therefore still matches and `RequireNoStale` stays quiet.
Decision record D2 asks for an allowlist "keyed `<file>` with a reason" that "reports its stale entries", and the list's own comment says an entry admits findings.
The list is empty today, so nothing is wrong yet, but the branch also has no test row: the production test passes an empty list and every fixture row passes `NewAllowlist(nil)`, so deleting line 114 breaks nothing.
The links scan does not have this problem, because it looks the key up only when a call matches.
Origin: the stale defect is from the code-correctness lens and the missing test row from the code-tests lens, kept as one finding because one change fixes both.

## Rejected

- Function-value use of `os.Symlink` (`f := os.Symlink; f(a, b)`) is missed (code-correctness F3): D4 states the scan reports calls, and the background file's description of the scan, "reports each `os.Symlink` or `os.Link` call", is accurate.
- `pattern/PATTERN-fs-links.md` last bullet makes a Windows statement (code-affected-set F2): D3's rationale says the background file points at `internal/fabricengine/doc.go`, and the bullet claims only that the path is unsettled, which that doc records; the "no Windows claim" criterion applies to the `PATTERN.md` entry, which makes none.
- `pattern/PATTERN-fs-links.md` line 3, "owns the platform difference between link kinds" (code-affected-set F3): describes what `internal/fslink` does, not that a Windows junction behaves correctly, so it is no claim of the kind D3 rules out.
- A call such as `exec.Command("claude", "-p")` under `claudeengine/` is reported twice (code-correctness note): the extra report changes only the count of failure lines and both are true findings.
- `claude --version && tmux display -p` reads as a headless line, and a `claude;` token is missed (code-correctness notes): both are inside D2's stated token rule and static-shape bound.
- The new `allowedSpawners` entry in `cmd/lym/tierpurity_test.go`: needed because the fixture rows hold `exec.Command` as source text, and its reason is accurate.
- Test economy: no fixture row duplicates another; each of the call-shape rows reaches a different `exprNamesClaude` branch.
