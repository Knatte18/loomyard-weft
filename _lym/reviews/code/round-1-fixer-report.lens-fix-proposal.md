```yaml
lens: fix-proposal
outcome: reported
```

No findings against the F1 proposal itself. One MEDIUM finding on the fix's coverage: it leaves `PATTERN-shed-driver` unmarked, so the new `doc.go` text is false for that cluster.

**MEDIUM — `internal/pattern/doc.go` (the added sentence) against `pattern/PATTERN-shed-driver.md`**
- The new text says "the cluster's background file marks each member rule that is review-only."
- `PATTERN.md` gives `PATTERN-shed-driver` the `(test)` marker.
- Its background file has four `Rule:` lines, at 7, 106, 110 and 114. I counted the `Rule:` lines and didn't open the file. Lines 106, 110 and 114 lack the `(test)` suffix, and the file holds no `Held by review alone` line.
- So that cluster has review-only members the file doesn't mark. The F1 fix covers every other cluster the review lists, but not this one.
- The review's location list omits it too, but the fix's own rule, "each background file of a cluster that carries `(test)` and has a review-only member," covers it.
- Evidence: `grep -c "^Held by review alone" pattern/PATTERN-shed-driver.md` returns 0, and the `PATTERN.md` entry carries `(test)`.
- Fix: add the same line under the three unmarked `Rule:` lines, or narrow the `doc.go` wording.

**Checks that came back clean**
- Challenging the proposal itself, I looked for a member the fix marks "held by review alone" whose own section names a test.
- The test references in the six files I checked fall outside the marked sections:
  - `PATTERN-stencil-ownership.md` lines 45 and 58 sit in the Parent directive and later sections, not in "Stencil ownership" or "Engine blocks".
  - `PATTERN-fabric-boundary.md` lines 15–26 belong to the vocabulary section, not "Git through fabric" or "Fabric isolation".
  - `PATTERN-gate-slot.md` lines 17 and 84 sit outside "Gate fixer", which starts at line 89.
  - `PATTERN-lym-files.md` lines 49–77 belong to the temp-file and link sections, which carry `(test)`.
- The other test-carrying clusters have no unmarked `Rule:` lines (`cli-cobra`, `cwd-resolution`, `refusal-way-forward`, `shuttle-provider-seam`, `sole-parsers`, `test-isolation`, `test-speed`, `tolerant-records`), so they need nothing.
- `github-auth` has no background file in `pattern/`, so there is nothing to mark there.

Calls made:
1. Bash, awk and grep over the held sections of the six files, to check for test references. My context didn't hold the sections beyond the `Rule:` lines.
2. Bash, grep for `Test[A-Z]`, `_test.go` and similar across the six files, to place each test reference by line. Same reason.
3. Bash, grep of `PATTERN-shed-driver.md` and the other `(test)` clusters, to find unmarked review-only rules. My context didn't hold those files.
