---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/gatecli/redfirst.go#embedPatternMatches
    summary: A directory embed pattern matches dot- and underscore-named descendants that Go excludes without all, so a new test-only fixture there is deleted on the base and the test's build failure counts as red
    fix: "In internal/gatecli/redfirst.go, keep the all: prefix as a flag in embedPatterns instead of discarding it (return each pattern with whether it carried all:). In embedPatternMatches, when the match is on an ancestor directory of below rather than on below itself, reject the match if any path element of below after the matched directory begins with . or _, unless the pattern carried all:. Add rows to TestBaseOverlay_ReplacesTheFilesOfTheModuleTheBasesBuildReads in internal/gatecli/redfirst_test.go: a plain directory pattern (for example add a directive embedding a directory named plain to sub/a/a.go) with sub/a/plain/.hidden.json and sub/a/plain/_x.json left alone, and sub/a/embedded/_x.json under all:embedded mapped to a copy."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redfirst.go:367
    summary: A changed non-Go file that a production file embeds from a testdata, dot- or underscore-prefixed directory keeps its working-tree text, and gatecli/doc.go states that an embedded file lies outside testdata, which Go does not require
    fix: "In baseOverlay in internal/gatecli/redfirst.go, for a non-Go file test only that it lies inside the module (dir equals module.rel or has the prefix module.rel/, or module.rel is .) instead of calling isModulePackage, and let embeddedAtBase decide; keep isModulePackage for .go files. Start embeddedAtBase's walk from the file's directory as now. Delete the line \"An embedded file lies outside `testdata`.\" from internal/gatecli/doc.go. Change the row NonGoFileUnderTestdataIsLeftAlone so it still expects no replacement (nothing embeds it), and add a row where sub/a/a.go embeds testdata/golden.json and sub/a/testdata/golden.json maps to a copy."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/redfirst.go#embedPatterns
    summary: embedPatterns splits a quoted pattern holding a space, misses a directive separated by a tab, and no test row covers its backtick-quoted form
    fix: "In embedPatterns in internal/gatecli/redfirst.go, match the directive as //go:embed followed by a space or a tab, then read its arguments the way Go does: a field opening with a double quote or backtick runs to its closing quote and is unquoted with strconv.Unquote, and an unquoted field ends at whitespace; then strip all:. Add to sub/a/a.go's directives in TestBaseOverlay_ReplacesTheFilesOfTheModuleTheBasesBuildReads a backtick-quoted pattern and a double-quoted name holding a space, with a row for each that expects a copy of the base text."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redfirst.go#embeddedAtBase
    summary: No test reaches embeddedAtBase's failure to list or read a base Go file, or its propagation out of baseOverlay
    fix: "In internal/gatecli/redfirst_test.go, add a test-only redundancyTree whose goFiles or file returns a fixed error, and add rows to TestBaseOverlay_ReplacesTheFilesOfTheModuleTheBasesBuildReads (or one table of its own beside it) that call baseOverlay with a changed non-Go file and assert a non-nil error naming the directory or file, and a nil replacement map, for each of the two failures."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomshed/doc.go:178
    summary: The loomshed package doc still says the unjudged exit comes for a hub or a diff the check cannot judge, though only a hub whose scoper is not go now produces it
    fix: "In internal/loomshed/doc.go line 178, replace \"for a hub or a diff the check cannot judge\" with \"for a hub whose scoper is not go\"."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/redfirst.go:189
    summary: The redFirstCall.run doc comment says it lays the base's production files over the working tree and omits the embedded non-Go files the overlay now restores
    fix: "In internal/gatecli/redfirst.go line 189, change the first line of the run doc comment to \"run lays the base's production files, and the non-Go files they embed, over the working tree, runs the changed tests and prints the verdict.\""
---

### [MEDIUM] Directory embed patterns match descendants Go excludes

**Location:** internal/gatecli/redfirst.go#embedPatternMatches
**Issue:** Go embeds a named directory's subtree except files and directories whose names begin with `.` or `_`, unless the pattern carries `all:`.
`embedPatterns` strips `all:` at `redfirst.go:438`, and `embedPatternMatches` accepts any ancestor of the file, so `pkg/assets/.fixture.txt` counts as embedded by a plain `//go:embed assets`.
When the diff adds that file and a new test embeds it, the base lacks it, so `baseOverlay` maps it to `""` and deletes it.
The test package then fails to build on the base, which `judgeRedFirst` counts as red, and the gate passes with no proof.
That is the false pass the change description's own design decision rules out for a fixture only a test embeds.
The layout is rare, so the severity stays below BLOCKING.

### [LOW] Embedded files under testdata and dot- or underscore-prefixed directories are never restored

**Location:** internal/gatecli/redfirst.go:367
**Issue:** `baseOverlay` skips any file whose directory fails `isModulePackage` (`internal/gatecli/redundancytests.go:191-193`) before asking `embeddedAtBase`.
Go allows a production file to embed `testdata/golden.json` or `_static/index.html`, so such a changed file keeps its fixed text on the base.
The changed test then passes there and the call exits with the green code, which sends the run back to a waiver.
This fails safe: it never passes the gate without proof.
The new line in `internal/gatecli/doc.go` "An embedded file lies outside `testdata`" states a rule Go does not have.
No production embed in this repository uses such a path today.

### [LOW] The directive scan misreads some valid directives

**Location:** internal/gatecli/redfirst.go#embedPatterns
**Issue:** `strings.Fields` splits `"my file.txt"` into two fields that match nothing.
The constant `goEmbedDirective` requires a space after `//go:embed`, so a tab-separated directive is missed.
Both cases leave an embedded file at its working-tree text, which fails safe toward the green exit.
The backtick in the trim set has no test row, so a mutant dropping it survives.

### [LOW] Base-tree read failures during embed discovery are uncovered

**Location:** internal/gatecli/redfirst.go#embeddedAtBase
**Issue:** `embeddedAtBase` returns an error when `base.goFiles` or `base.file` fails, and `baseOverlay` passes it on.
Those reads go through git via `commitTree`.
`memoryTree` in `redfirst_test.go` never fails, and the overlay table always expects no error, so no test pins either path.

### [LOW] loomshed package doc keeps the removed cause of the unjudged exit

**Location:** internal/loomshed/doc.go:178
**Issue:** The doc says exit `gateslot.RedFirstUnjudgedExit` fails "for a hub or a diff the check cannot judge".
After this diff only the scoper check in `runRedFirst` returns it, as `gateslot/redfirst.go` and the `redFirstFailure` text in `loomshed/redfirstgate.go` now say.

### [NIT] redFirstCall.run's doc omits the embedded files

**Location:** internal/gatecli/redfirst.go:189
**Issue:** The file header at `redfirst.go:3` and `baseOverlay`'s doc name the non-Go files the overlay restores, but `run`'s doc comment still says it lays only the base's production files over the working tree.

## Holistic notes

The diff makes the change the Goal asks for: the motivating file `contracts/specs/raddle-proposal-spec.md` is embedded by name from `contracts/specs/specs.go`, so the new overlay restores it, and the unjudged refusal for non-Go files is gone.
Every change traces to the Goal or to the parent's added entry, which licenses the stencil edit; that edit drops only the removed cause and keeps the rest of the rule.
The `Root-Cause:` trailer of `4abc6f41` names the blanket classification in `baseOverlay` that the diff removes.
`redFirstCall.run` still yields exit 0 for red, the green code when every tier passes and 1 for a run that judged none; exit 5 comes only from the scoper check.
No base assertion is loosened: the old refusal's assertions were replaced because the behaviour they pinned was removed, and the loomshed row now pins the full exit-5 sentence.
The tests' expected values are literals, so none is tautological, and the new integration row shows with the real go command that the overlay replaces an embedded file.
A base Go file that does not parse yields no patterns from the line scan rather than an error; the build on the base then judges it, as for any base build failure.

## Rejected

- Assistant F1's claim that a directive-shaped line inside a raw string or block comment of a base production file can force a false pass: it needs production source holding such a line at a line start and a new test embedding the matched file, which is contrived; the parsing part of that finding is kept as F3 at LOW, since the missed cases fail safe toward the green exit.
- Assistant F1, F2 and F3 at BLOCKING: F2's and F3's cases fail safe and occur in no embed of this repository; F1's false pass needs an unusual layout; each is kept at the severity its own block gives.
- Code-correctness lens NIT on quoted patterns holding a space: merged into F3.
- Code-tests lens LOW on the uncovered backtick strip: merged into F3.
- Assistant F4 at MEDIUM: merged into F4 at LOW, since the propagation it covers is a plain return of a git read error.
