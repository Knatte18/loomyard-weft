---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: tools/deploy/main_test.go:187
    summary: lyxMainPackage is never checked against the real lyx main package, so a drifted constant silently disables the foreign-lyx warning
    fix: In tools/deploy/main_test.go add a source-level guard next to TestLdflagsPath_MatchesBuildinfoChannel, or a row in it, that reads go.mod's module path and asserts lyxMainPackage equals that module path plus /cmd/lyx. Keep it a file read, spawning nothing.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: tools/deploy/main.go:otherLyxBinaries
    summary: seenFiles keys are lowercased by normDir, so on a case-sensitive filesystem two distinct files differing only by case collapse to one key and a real stale lyx goes unreported
    fix: In otherLyxBinaries key seenFiles on the resolved path as returned by filepath.EvalSymlinks, with no normDir folding, at all three sites (seed, lookup, insert); on Windows EvalSymlinks already returns the on-disk casing. Keep normDir for the directory keys only, and revert its comment to say it compares PATH entries.
    origin: lens:code-correctness
---

### [LOW] lyxMainPackage has no test against the real main package

**Location:** `tools/deploy/main.go` (`lyxMainPackage`), `tools/deploy/main_test.go:187`
**Issue:** The new test passes the test binary's own build-info path as `mainPackage`, so nothing pins the production constant to `go.mod`'s module path plus `/cmd/lyx`.
A module rename or a typo would make the check match no lyx and the warning would never fire, with every test still green.
The package already guards the similar hard-coded `-X` path in `TestLdflagsPath_MatchesBuildinfoChannel`.
The constant and the behavior built on it are added by this diff, so the gap is the diff's.

### [LOW] seenFiles folds case on resolved file paths

**Location:** `tools/deploy/main.go` (`otherLyxBinaries`, the three `seenFiles[normDir(resolved)]` uses)
**Issue:** `normDir` lowercases and trims separators.
Applied to resolved file paths on a case-sensitive filesystem, `/opt/Tools/lyx` and `/opt/tools/lyx` share a key, and the second is skipped unreported.
The old code folded case on directory names only, so extending it to resolved files is new here.
Likelihood is very low and the failure is a missed warning, not a wrong deploy.

## Rejected

- code-tests F1, `pushedGate` has no test after `prodAdvanceError` and its test went: the refusal is retained behavior, not behavior the diff adds, so the rubric's no-coverage clause does not apply. The removed boolean-fed function decided nothing the git call did not already decide, and the package header records that its tests spawn no git.
- Focus items on `buildinfo.ReadFile` of a `deploy`-built lyx and on Windows `lyx.exe`: settled as not a defect. `go build ./cmd/lyx` records the main package path `github.com/Knatte18/loomyard/cmd/lyx`, and `debug/buildinfo` reads PE files, so `lyx.exe` is covered.
- Focus items on a stale symlink to a different file and on seeding `seenFiles` with the production binary: settled as not a defect. A link to a different file resolves to a new key and is reported, a link to the production binary is skipped, and a missing production binary only leaves the seed empty.
- Focus item on leftover `prod` references: none remain in code, comments, package header, `update-plugins.*`, `docs/` or `PATTERN.md`. The one hit, `docs/research/creel-audit-2026-10-16/report.md:81`, is a dated research record and not a claim about the script. The orch rules on Bolt are outside the diff per the board entry.
- Summary claims checked against the diff: the `pushedGate` rationale, each file once, and no report of the production alias all hold.
