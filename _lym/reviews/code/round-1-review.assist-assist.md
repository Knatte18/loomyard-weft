---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redfirst.go:433
    summary: The line scanner misreads valid embed directives and accepts directive text inside strings or block comments.
    fix: Replace the line scan in embedPatterns with recognition of actual Go directive comments and Go-compatible argument parsing, preserving quoted spaces, escapes and valid whitespace separators. Add regression rows to the existing overlay test for quoted filenames, tab separators and directive-shaped text inside strings and block comments.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redfirst.go:367
    summary: Package-discovery filtering leaves production-embedded assets in hidden directories and testdata at HEAD.
    fix: In baseOverlay, separate module containment from package-directory filtering for non-Go files. Evaluate production embed patterns for assets beneath testdata and dot- or underscore-prefixed directories, retaining HEAD only when production does not embed them. Add rows distinguishing production-embedded assets from test-only assets in these directories and correct the associated documentation's blanket testdata exclusion.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redfirst.go:446
    summary: Directory matching includes hidden descendants that Go excludes and can falsely pass red-first by deleting a test-only fixture.
    fix: Preserve the all-prefix information in embedPatterns and make embedPatternMatches distinguish direct glob matches from directory traversal. Exclude dot- and underscore-prefixed descendants during ordinary directory traversal, but include them when all applies. Add existing-table rows for ordinary and all directory patterns and a regression in the existing integration test where production embeds assets and a new test alone embeds assets/.fixture.txt.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redfirst.go:405
    summary: The new base-directory and production-source read failures have no regression coverage.
    fix: Extend the existing overlay behavior test with a test-only tree reader that can fail goFiles or file during embed discovery. Assert the contextual error and no usable replacement map for each failure, including an ancestor-directory lookup.
---

### [BLOCKING] Parse real embed directives with Go argument semantics

**Location:** internal/gatecli/redfirst.go:433
**Issue:** A directive naming `"a b.txt"` is split into `a` and `b.txt`, leaving the production asset at HEAD.
A test expecting the fixed contents then passes on the purported base.
The literal prefix also misses a tab separator.
Conversely, directive-shaped lines inside a raw string or block comment are accepted.
They can cause a new test-only embedded fixture to be deleted and turn its build failure into false regression evidence.
The installed Go contract allows quoted filenames, and its scanner distinguishes comments from string contents.

### [BLOCKING] Classify assets independently of package discovery

**Location:** internal/gatecli/redfirst.go:367
**Issue:** With production `//go:embed all:assets`, Go embeds `assets/_private/data.txt`.
The call to `isModulePackage` rejects its directory before embed discovery runs.
The same filter rejects dot-prefixed directories and explicitly embedded `testdata` assets.
These changed production inputs retain fixed contents, so a real regression test can remain green on the base.
The filter's implementation is in `internal/gatecli/redundancytests.go:183`.

### [BLOCKING] Preserve hidden-descendant semantics

**Location:** internal/gatecli/redfirst.go:446
**Issue:** Let production embed `assets`, containing an ordinary file, and a new test explicitly embed `assets/.fixture.txt`.
Go excludes that hidden descendant from the production directory embed.
The new matcher nevertheless matches its ancestor `assets`.
Because the base lacks the fixture, the overlay deletes it, causing the test package to fail to build.
`judgeRedFirst` counts that failure as red, and the gate passes without evidence that the test reaches the bug.
Removing `all:` at line 438 discards the distinction Go's directory traversal requires.

### [MEDIUM] Cover failures during embed discovery

**Location:** internal/gatecli/redfirst.go:405
**Issue:** The new directory-list and production-source reads can fail, but both methods of the test's `memoryTree` always succeed.
The overlay table always expects no error.
No changed integration row exercises either failure, so the new propagation paths lack an assertion.

## Consolidation

The tests lens's parsing and inclusion coverage finding is folded into F1–F3.
The affected-set and scope lenses returned no findings.
No proposed finding was rejected.

The exit-5 producer and consumer now name the scoper alone.
Existing real-Go and gate tests exercise red, green, no-test and unsupported-scoper outcomes despite map reach limitations.
The root-cause trailer identifies the removed blanket refusal.
The parent amendment licenses the stencil edit, which preserves the red-first obligation and parent recovery procedure.
Documentation removes the old non-Go refusal; its asset-selection claims need the corrections above.
The changed tests use independent expected values and add no waits or sleeps.
Replacing the old refusal assertions implements the requested behavior rather than weakening an unchanged check.

For malformed base Go source, the scanner produces no parse error; the later build judges it.
A base build failure counting as red is the existing documented contract, so this is no additional finding.

## Validation

Attempted `timeout 3600s lym gate test --tier 0 ./internal/gatecli -- -run '^TestBaseOverlay_'`.
The sandbox refused writes to the hub's gate-holder and run-process records before a tier ran.
No test result is claimed for this review attempt.
The findings are grounded in the changed code, its callers and the installed Go embed implementation.
