---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/testcover/testselect.go:424
    origin: lens:code-correctness
    summary: "embedPatterns reads only non-test Go files, so an inert file that the importing package's own _test.go go:embeds is treated as inert and an edit to it narrows the run, which D5 forbids"
    fix: "In internal/testcover/testselect.go, make embedPatterns also read the working tree's _test.go files when the package at dir is the importing package (ownFilesMiss path), while a linked package keeps reading non-test Go files only. Pass a flag or the file filter from ownFilesMiss and linkedChanges, and key the embeds cache by that choice. Add a TestSelectorSelect row where b/b_test.go go:embeds an inert file, the file changes, and the want is MissNonGoFile."
  - id: F2
    severity: MEDIUM
    class: decision
    location: internal/testcover/testselect.go:129
    origin: handler
    summary: "selectTests selects every spawn-marked test whenever any linked function changed; the decision record does not say this and D4 criterion says no B test that covered only other functions is selected"
    fix: "Leave the code in place and defer this finding to the run's parent: the sound change is recording the spawn-marked rule with lym loom decision add, or refusing it, and that lies outside the fixer's write surface. Do not revert the selection."
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/testcover/testselect_test.go:152
    origin: lens:code-tests
    summary: "no row edits a method body, so TestSize is never selected and the Type.Method key naming that the selector matches against the map's block keys is unpinned"
    fix: "Add a TestSelectorSelect row that edits the body of Box.Size in a/a.go (for example replace 'return b.N' with 'return b.N + 1') and wants TestSize and TestSpawn."
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/testcover/testselect_test.go:175
    origin: lens:code-tests
    summary: "several listed D5 miss and no-miss cases have no row: BuiltAt-only and Version-only marker mismatch, parse failure at the map's commit, probe file absent at the commit, removed const or var or type, package clause change, removed init, implicit-value const group without iota, removed go: line, predeclared name added as var or const or type, a method named max, removed method, changed signature with the same receiver, own Go file added or removed, own non-Go file changed, recorded non-Go file removed"
    fix: "Add rows to TestSelectorSelect for each case named in the summary, each as a one-line linkedEdits, files or editMap entry; use editMap to set Format.BuiltAt and Format.Version, a commit fixture that lacks b/b.go for the absent probe, and a fixture edit that makes the committed copy of a/gone.go unparseable for the commit-side parse failure. A removed init must want TestInit and TestSpawn; a method named max must be no miss."
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/testcover/testselect.go:415
    origin: lens:code-tests
    summary: "go:embed pattern handling is pinned only for one plain directory pattern; quoted and backquoted patterns, the all: prefix, several patterns on one line, globs, a malformed line and an unparseable Go file in embedPatterns have no row"
    fix: "Add TestSelectorSelect rows where a/assets.go carries each directive shape (quoted, backquoted, all:static, two patterns, a glob such as static/*.txt) with an inert page.txt edit wanting MissNonGoFile, a malformed directive line wanting MissParse, and an inert edit under an unrelated directory wanting no miss."
  - id: F6
    severity: LOW
    class: design
    location: internal/testcover/declcompare.go:80
    origin: lens:code-correctness
    summary: "deleting a linked Go file that imports anything gives MissNonBody for the removed import, so the D4 criterion that a deleted file selects the tests that covered it holds only for import-free files"
    fix: "In compareGoSources skip the removed-import check when after.present is false, since imports are file-scoped and a deleted file's imports reach no other file; keep the const, var, type, directive and method checks as D5 words them. Add a row where a/gone.go imports strings, is deleted, and wants TestH and TestSpawn."
  - id: F7
    severity: LOW
    class: design
    location: internal/testcover/testselect.go:393
    origin: lens:code-correctness
    summary: importAllowed treats any import path whose first segment has no dot as standard library, so a dotless non-stdlib module path is wrongly allowed
    fix: "Replace the first-segment test with an explicit standard-library check, for example go/build's IsStandardImport-style rule over the toolchain's GOROOT/src listing, and add a row where an added import of a dotless non-module path wants MissNonBody."
  - id: F8
    severity: LOW
    class: consistency
    location: internal/testcover/testselect.go:37
    origin: handler
    summary: "doc comments are written in terms of other named symbols: MapsDir 'as Dir returns it', workingHash and changedFiles 'ContentHash', goFileComparison.changed 'as a block key does'"
    fix: "Rewrite each comment to stand alone: testselect.go:37 name the directory as the one holding the per-package coverage maps; testselect.go:110 and :274 say 'content hash'; declcompare.go:32 say 'by the name the coverage map's function keys use, Type.Method for a method'. Keep the self-reference to the symbol's own name."
---

### [MEDIUM] Own test file embeds are invisible to the inert carve-out

**Location:** internal/testcover/testselect.go:424
**Issue:** `embedPatterns` skips `_test.go` files. For a linked package that is right, since its test files are not in the importing binary. For the importing package, `ownFilesMiss` runs the same carve-out over its own files, whose test files are in the binary. An inert file that `b_test.go` embeds therefore changes with no miss, and the run narrows, against D5's rule that misses widen and never narrow.

### [MEDIUM] Spawn-marked tests are selected without a recorded decision

**Location:** internal/testcover/testselect.go:129
**Issue:** `selectTests` adds every test whose spawn verdict is out of process or unresolved whenever any linked function changed. The change description names this a design decision and says the record does not state it. It widens, so it is safe, but D4 and its first acceptance criterion select only tests that covered a block of the changed function. Per the rubric this is an unrecorded deviation, filed for the parent to record or refuse.

### [MEDIUM] Method bodies are never exercised by the selector test

**Location:** internal/testcover/testselect_test.go:152
**Issue:** The fixture map holds `TestSize` on `Box.Size`, but no row changes that method, so the `Type.Method` naming that the selector shares with the map's block keys is unpinned; breaking it would not fail the test.

### [MEDIUM] Listed D5 cases without a row

**Location:** internal/testcover/testselect_test.go:175
**Issue:** The record's acceptance criteria name a marker whose `Commit` and `BuiltAt` differ; only `Commit` is edited. Parse failure is pinned on the working-tree side only. The other cases named in the summary are branches the code adds with no row that would fail if they broke. They are one gap, a table that stops short of D5's list, filed once.

### [MEDIUM] go:embed pattern parsing has one row

**Location:** internal/testcover/testselect.go:415
**Issue:** The carve-out is a D5 item with its own criterion, and the code behind it parses quoted, backquoted, `all:`, multiple and glob patterns. Only a plain directory pattern is pinned, so a regression in the rest passes the suite and narrows the run.

### [LOW] A deleted file with imports runs the package whole

**Location:** internal/testcover/declcompare.go:80
**Issue:** A deleted file has no working-tree imports, so every import it held counts as removed and gives `MissNonBody`. The `DeletedFile` row passes only because its fixture file imports nothing. Most real files import something, so the D4 deletion criterion mostly yields a miss. Safe, but it departs from the criterion's reading.

### [LOW] Dotless non-stdlib import paths count as standard library

**Location:** internal/testcover/testselect.go:393
**Issue:** `importAllowed` allows any path whose first element has no dot. A `replace`d or workspace module with a dotless path is allowed where D5 asks for a miss.

### [LOW] Doc comments lean on other symbols

**Location:** internal/testcover/testselect.go:37
**Issue:** The comment rules ask for a comment that is correct to a reader who knows no other symbol. The named comments refer to `Dir`, `ContentHash` and block keys.

## Rejected

- Lens code-tests item 14, `NestedPackageDirs` has no direct test: the function moved unchanged and the diff adds no behavior; `NestedPackageFile` pins the selector's use of it, and the old location had no test either.
- Lens code-tests items 13 and 15, cache behavior, `Select` called twice, and the content of `Detail`: internal caching and message wording are not public-surface behavior, and a test for them would be the per-helper test the rubric flags.
- Lens code-tests items 1, 6, 7 and 12 as separate findings: folded into the single missing-rows finding F4, since they are one gap in one table.
- Lens code-tests items 2 to 5, 8 and 9 as separate findings: items 2 stays as F3 for the focus on method naming; the rest fold into F4.
- Lens code-affected-set: it raised no finding, and its set check agrees with mine: the move left no stale reference, and `doc.go` and `docs/overview.md` match the behavior.
- Review-range question on commit messages: neither commit carries a `Pattern-Departure:` trailer and none is needed.
