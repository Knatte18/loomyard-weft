---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/burlerengine/template_test.go#TestTemplate_FillsWithAllMarkers
    summary: The two template tests the redundancy report lists as contained by TestAnswerParentAsk assert what it never does, yet carry no testtiming keep marker, as PATTERN-test-speed requires of a kept candidate
    fix: Add a line `//testtiming:keep pins the explore, review and fix steps' per-marker missing-value errors, which TestAnswerParentAsk never renders` directly above `func TestTemplate_FillsWithAllMarkers` in internal/burlerengine/template_test.go, and a line `//testtiming:keep pins that each optional explore and fix marker renders empty cleanly and lands in its place, which TestAnswerParentAsk never renders` directly above `func TestTemplate_OptionalDirectives`. Keep both tests unchanged otherwise.
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/burlerengine/diffpieces.go:144
    summary: The piece cap counts bytes only, so a piece under 60,000 bytes can still pass the agent read's default line limit or its token limit and fail to read whole, the failure the board entry's Seen section records
    fix: In internal/burlerengine/diffpieces.go add a named constant diffPieceMaxLines (1,500, under the 2,000-line default read) and make boundedUnits and packUnits close a unit or text when either its byte length passes diffPieceMaxBytes or its newline count passes diffPieceMaxLines; lower diffPieceMaxBytes to 40,000 so dense content such as a --binary base85 patch stays under about 25,000 tokens. Update the constant's doc comment, the burlerengine doc.go paragraph and the summary's 60,000 figure to match, and add a TestWriteDiffPieces row whose many short lines pass the line cap but not the byte cap and must split.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: contracts/stencils/burler/burler-diff-pieces.md:55
    summary: The stencil calls the pieces the diff under review and has them read in place of the range's git diff, but for a merge-parent task (a darn run, whose row now stages) or a none base the rubric's range is not that patch, and the stencil never defers to the rubric
    fix: In contracts/stencils/burler/burler-diff-pieces.md replace the opening sentence with one saying the pieces hold the task's full diff against its merge base, written out below in pieces of one package each, and add a sentence after the read-in-place line saying that where the rubric narrows or refuses the review range, as for a merge from the parent or an unreadable base, the rubric's range governs and the pieces serve only as reading aid for the commits that range names.
    origin: lens:code-affected-set
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/shedrecipe/entries_burler.go:62
    summary: The staged patch is the working tree against the base plus untracked files, not the committed base..HEAD range the rubric defines, so on a dirty tree the reviewer reads uncommitted or untracked content as the subject
    fix: Stage a committed patch instead. Add to internal/fabricengine a reader of `git diff --no-color --no-ext-diff --no-textconv --no-renames --binary -U3 <base> HEAD`, expose it to shedrecipe through a code-subject-style seam on Env read per call (wired in internal/loomcli/wiring.go from taskMergeBase), and have subjectDiff use it in place of Subject.Patch; leave the Bouncer's CodeSubject patch unchanged. Extend StageDiffGivesTheCodeSubjectsPatch to assert the committed patch reaches profile.Diff, and add a fabricengine row with a committed change plus an untracked file asserting only the committed change is in the patch.
    origin: assist:assist
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/burlerengine/diffpieces_test.go:392
    summary: Two promised behaviours have no failing row - the hunk-first then line-end split order (any byte slicing under the cap passes the join check) and the assistant's explore step naming the same pieces
    fix: In TestWriteDiffPieces replace the wantJoined check of FileOverTheCapSplitsAtHunksAndLines with exact wantContents per piece, choosing maxBytes so the first piece ends at the second hunk's `@@ ` line and an oversized hunk breaks only at a line end. In the existing assisted-round test of internal/burlerengine (the one that runs Review with an assistant seat) set p.Diff to a two-package diff and assert both the chair's and the assistant's instruction-1-explore.md name the same backticked piece paths.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/shedadapters/burlerreview_test.go:778
    summary: Several new paths have no row - writeDiffPieces's directory and write failures, formatDiffPieces's root and part labels, subjectDiff's error pass-through, and the reader's one call per spawning call shared by a retried attempt
    fix: Add rows to the existing tests rather than new tests. TestWriteDiffPieces gets a row whose dir sits under a regular file and asserts a non-nil error; TestEngine_Review_StagesTheDiff's DiffIsStaged gets a root file and a split package and asserts the explore step carries `(the repository root)` and `part 1 of 2`; StageDiffGivesTheCodeSubjectsPatch gets a row whose subject returns an error and asserts the spawned profile's Diff is empty; the diffRows of TestBurlerReviewProducer_Call_ProfileAndOpts get a counting reader over a died-then-done runner asserting one read and the same Diff on both attempts.
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/burlerengine/diffpieces.go:243
    summary: When one file is split across pieces, every piece after the first starts with no diff --git or +++ header, so read alone its hunks cannot be tied to a file
    fix: In boundedUnits, take the file's header (the text before its first `@@ ` line) and prefix it to every unit after the first, counting it against maxBytes; in formatDiffPieces or the unit itself mark a continued file, for example by keeping the header line unchanged. Pin it in the FileOverTheCapSplitsAtHunksAndLines row with exact piece contents.
    origin: lens:code-correctness
  - id: F8
    severity: LOW
    class: design
    location: internal/burlerengine/diffpieces.go:230
    summary: diffFilePackage takes the last space-b-slash of the header, so a new-side path that itself holds space-b-slash (docs/foo b/bar.md) is grouped under the wrong package
    fix: Read the new-side path from the section's `+++ b/` line (or `+++ /dev/null` falling back to the `--- a/` line for a deletion) instead of the `diff --git` header, unquoting a C-quoted path with strconv.Unquote. Add a TestWriteDiffPieces row with a path `docs/foo b/bar.md` asserting its package is `docs/foo b`.
    origin: lens:code-correctness
  - id: F9
    severity: LOW
    class: design
    location: internal/burlerengine/diffpieces.go:199
    summary: pieceFileName folds the whole package path into one file name, which passes a 255-byte name limit for a deep package and fails Review before any reviewer starts
    fix: In pieceFileName cap the flattened package part at a named length such as 80 bytes, cutting from the left so the package's last directories stay readable; the seq prefix keeps names unique and diffPiece.Package keeps the full path for the listed bullet. Add a TestWriteDiffPieces row with a package path over 300 bytes asserting the piece is written.
    origin: assist:assist
  - id: F10
    severity: NIT
    class: design
    location: internal/burlerengine/diffpieces.go:255
    summary: An over-cap line is cut at a byte offset, which can split a multibyte UTF-8 rune across two pieces
    fix: In boundedUnits, before cutting line[:maxBytes], step the cut back while utf8.RuneStart(line[cut]) is false, and cut there.
    origin: lens:code-correctness
  - id: F11
    severity: NIT
    class: consistency
    location: internal/burlerengine/template_test.go:91
    summary: TestTemplate_FillsWithAllMarkers's doc still says only pattern_directive and focus_directive are excluded from the deletion sweep, while the diff adds diff_pieces to its optional list
    fix: Rewrite the two lines above `func TestTemplate_FillsWithAllMarkers` to say the explore step's optional markers are excluded from the deletion sweep, since deleting an optional marker must not error, without naming them.
    origin: lens:code-affected-set
---

### [BLOCKING] Contained template tests lack keep markers

**Location:** internal/burlerengine/template_test.go#TestTemplate_FillsWithAllMarkers

**Issue:** The redundancy report lists `TestTemplate_FillsWithAllMarkers` and `TestTemplate_OptionalDirectives` as contained, covered by `TestAnswerParentAsk`.
`TestAnswerParentAsk` (`internal/burlerengine/parentreview_test.go:143`) asserts parent-ask states and report contents, never a stencil marker's error or placement, so both template tests pin what coverage cannot see and are kept.
`PATTERN-test-speed` says a kept candidate carries `//testtiming:keep <reason>` directly above its `func Test…` line; neither does (`template_test.go:93`, `:182`).
A violation of a listed constraint is BLOCKING.

### [MEDIUM] Byte-only cap does not guarantee a whole read

**Location:** internal/burlerengine/diffpieces.go:144

**Issue:** `boundedUnits` and `packUnits` measure `len()` only.
An agent read stops at 2,000 lines by default and refuses past about 25,000 tokens.
60,000 bytes of short lines is well over 2,000 lines, and a dense `--binary` base85 patch (`taskpatch.go:84`) runs near 2 bytes per token before line-number prefixes.
Either case leaves the reviewer with a truncated or refused piece, the very failure the board entry records.

### [MEDIUM] Diff-pieces stencil overrides the rubric's range

**Location:** contracts/stencils/burler/burler-diff-pieces.md:55

**Issue:** The stencil says "The diff under review is already written out below" and "Read these pieces in place of the review range's `git diff`".
The pieces are the code subject's patch against the task merge base (`internal/loomcli/subjects.go:153-181`).
For a merge from the parent, which `lym beam merge-parent` files as a darn task (`internal/beamcli/mergeparent.go:76`), the rubric narrows the range to the merge commit (`loom-rubric-code-review.md:38-40`), and darn's Code-Burler-Review row now stages.
For a `none` base the rubric orders a BLOCKING finding and no review.
The stencil asserts the pieces are the subject in both cases, and the summary's claim that these cases "still go through its own rules" is not borne out by the text.

### [MEDIUM] Staged patch is the working tree, not the committed range

**Location:** internal/shedrecipe/entries_burler.go:62

**Issue:** `subjectDiff` hands over `Subject.Patch`, which `ReadTaskPatch` renders as `git diff <base>` against the working tree plus one new-file diff per untracked path (`internal/fabricengine/taskpatch.go:83-99`).
The rubric's subject is `git diff <base>..HEAD`, the committed diff, and the stencil tells the reviewer to read the pieces in its place.
In a run the tree is normally committed before the review row, which keeps this from BLOCKING, but an untracked file or an uncommitted edit becomes the reviewed subject with nothing to flag it.

### [MEDIUM] Split order and assistant pieces are unpinned

**Location:** internal/burlerengine/diffpieces_test.go:392

**Issue:** The two oversized rows assert only the cap, the joined text and that a split happened; cutting at arbitrary byte offsets passes them, so the hunk-first then line-end rule the doc comment promises has no failing row.
The assistant's explore step naming the same pieces is claimed in `doc.go` and the summary and holds through `assistProfile` copying `*p` (`assist.go:83-90`), but no assisted test sets `Diff`.

### [LOW] Failure, label, error and once-per-call paths have no row

**Location:** internal/shedadapters/burlerreview_test.go:778

**Issue:** No row reaches `writeDiffPieces`'s `MkdirAll` or `WriteFile` failure, `formatDiffPieces`'s `(the repository root)` or `part N of M` labels, `subjectDiff`'s error return (`entries_burler.go:62-66`), or the reader being called once and shared by a retried attempt (`burlerreview.go:376`).

### [LOW] Continuation pieces of a split file have no header

**Location:** internal/burlerengine/diffpieces.go:243

**Issue:** `boundedUnits` cuts a large file into its header chunk and its hunks, and `packUnits` may put later hunks in a new piece.
That piece opens with a bare `@@` hunk, or mid-hunk after a line-end cut, with nothing naming the file it belongs to.

### [LOW] Package parse misreads a path holding space-b-slash

**Location:** internal/burlerengine/diffpieces.go:230

**Issue:** For `diff --git a/docs/foo b/bar.md b/docs/foo b/bar.md`, `LastIndex(" b/")` lands inside the new path and returns `.`, so the file is grouped and labelled at the root.
Stripping quotes also leaves git's octal escapes undecoded.
The bytes still land in a piece.

### [LOW] Flattened package name is unbounded

**Location:** internal/burlerengine/diffpieces.go:199

**Issue:** `pieceFileName` replaces every `/` with `-`, so the file name grows with the package path.
A path over the filesystem's 255-byte name limit makes `os.WriteFile` fail, and `Engine.Review` returns that error before a reviewer starts.
Such paths are rare, which keeps this LOW.

### [NIT] Over-cap line cut can split a rune

**Location:** internal/burlerengine/diffpieces.go:255

**Issue:** `line[:maxBytes]` cuts at a byte offset, so a multibyte UTF-8 character can straddle two pieces, leaving each with invalid UTF-8 at the cut.

### [NIT] Stale doc on TestTemplate_FillsWithAllMarkers

**Location:** internal/burlerengine/template_test.go:91

**Issue:** The doc comment names only `pattern_directive` and `focus_directive` as excluded from the deletion sweep, while the test's optional list now holds four markers, `diff_pieces` among them.

## Rejected

- code-tests NIT, darn and sampler rows unpinned: a recipe row's `stage_diff: true` is static configuration, and the rubric's coverage bullet asks for behaviour, which `StageDiffGivesTheCodeSubjectsPatch` covers through the key.
- assistant F1 at BLOCKING: kept as F4 at MEDIUM, since the run commits before the review row and the dirty-tree case is a hazard, not the normal path; no listed constraint is broken.
- code-scope: no findings; every change traces to the board entry's two goal points or is forced by one.

Focus questions settled: the cap holds at every level in bytes (F2 covers the line and token gap); empty diffs write nothing; renames are moot under `--no-renames`; names are deterministic and the seq prefix keeps them unique; write failures are surfaced; the diff is read once per spawning call after the probe and hand-on exits, an unset key or absent subject gives none, and a failed read is logged; the explore step names the pieces and renders unchanged without them; only the three Code-Burler-Review rows stage; the diff makes no change outside the board entry's Idea.
