# Card 1 — shared-reference-delete-order

**Create:**
- `plan:internal/planglyph#memberReferences` -> `func memberReferences(plan *planparser.Plan, worktreeRoot string, subjects []coverageSubject, candidates []string, loader typesLoader) (resolved, nameOnly []map[string][]int, err error)`
- `plan:internal/planglyph#deleteOrderFindings` -> `func deleteOrderFindings(subjects []coverageSubject, resolved, nameOnly []map[string][]int, regions []editRegion) []Finding`
- `plan:internal/planglyph#laterImportFindings` -> `func laterImportFindings(lang glyph.Language, deleting []planparser.Card, regions []editRegion, worktreeRoot string) ([]Finding, error)`
- `plan:internal/planglyph#ignoredByGoTool` -> `func ignoredByGoTool(dirName string) bool`
- `plan:internal/planglyph#dispatchFindings` -> `func dispatchFindings(plan *planparser.Plan, worktreeRoot string, completed, forthcoming []planparser.Card) ([]Finding, error)`
- `plan:internal/planglyph#countingTypesLoader` -> `type countingTypesLoader struct`
- `plan:internal/planglyph#countingTypesLoader.load` -> `func (l *countingTypesLoader) load(root string, dirs []string) ([]*packages.Package, error)`

**Edit:**
- `internal/planglyph#LaterDeleteReferences`
- `internal/planglyph#callerCoverageFindings`
- `internal/planglyph#planGatePass`
- `internal/planglyph#loadTypedReferences`
- `internal/planglyph#candidateFiles`
- `internal/planglyph#resolvePass`
- `internal/planglyph#ValidateDispatch`
- `internal/planglyph#ValidateFormatAfter`
- `internal/planglyph#validateFormatAfter`
- `internal/planindex#Index.LaterDeleteReferences`
- `internal/planglyph#TestLaterDeleteReferences`
- `internal/planglyph#TestGlyphChain_CallerCoverage`
- `internal/planglyph#coverageFiles`
- `internal/planglyph#TestGlyphChain_CallerCoverageTyped`
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Delete:**
- `internal/planglyph#deletedSymbol`

**Intent:**
Make `delete-before-reference` find a deleted member's references through `caller-uncovered`'s reference computation instead of the bare-word regexes of `deleteorder.go` (Decisions 1–4, issue #562).

The shared computation:
`memberReferences` is the reference half of today's `callerCoverageFindings`, lifted out unchanged in meaning: it runs `loadTypedReferences` over `candidates`, scans every file the load did not check and every line it left unresolved with `scanReferences`, and merges each subject's lines with `combineReferences` into `resolved` (typed, or a package-level line the scan resolves by import path) and `nameOnly` (a method line matched by name alone).
`loadTypedReferences` takes a nil `loader` as "run no load" and returns `typedReferences(nil, …)`, so a scan-only caller passes nil and spawns nothing.
`callerCoverageFindings` calls `memberReferences` once over its whole-tree `candidateFiles`, then partitions each deleted subject's lines: a line inside a later card's Edit region (`insideLaterEditRegion`) goes to `deleteOrderFindings` whether or not an admissible target also covers it, as today, and only the rest go through `coveredByTarget` into `caller-uncovered`; a re-signed subject's lines all stay `caller-uncovered`'s, as today.
So one plan-gate pass runs one type load, shared by both checks, and a reference is reported by exactly one of them.
`deleteOrderFindings` emits, for a deleted subject, one blocking `delete-before-reference` finding per resolved line inside a later card's region, deduplicated per subject, file and line, with today's detail ("card N deletes …, but card M still references it at file:line in its Edit code — move the delete to a card after card M"), naming the first later card whose region holds the line; and one informational `delete-before-reference` finding per deleted method whose name-only lines fall inside later regions, naming every such file with its lines as `<file> at line <n>`, the ID of each later card whose region holds one of them, stating that the receiver could not be resolved, and naming the same way forward (Decision 2).
A line inside the subject's own declaration span is already dropped by `insideOwnSpan` in both the typed walk and the scan, so a later card editing the declaring file reports nothing for that line.

Only Go code is searched (Decision 3).
A member subject's references come only from tokenized `.go` candidates, so a comment, a string literal or a non-Go Edit target such as `docs/notes.md#` never matches.
`ignoredByGoTool` is the directory-name rule `candidateFiles` applies today (`testdata`, `vendor`, a leading `.` or `_`), and `candidateFiles` calls it.
`laterImportFindings` keeps a package self glyph Delete's quoted import-path match, blocking, but searches only later regions whose file ends in `.go` and has no directory element `ignoredByGoTool` rejects, and skips a region file absent on disk.

`planGatePass` appends `laterImportFindings` over the plan's own cards after `callerCoverageFindings`, so `ValidateFormat`, `Validate`, `ValidateRework` and `ValidateFormatAfter` over its pending cards all report the typed, partitioned form.
`resolvePass` no longer calls `LaterDeleteReferences`.
`dispatchFindings` is today's `ValidateDispatch` body; `ValidateDispatch` is `dispatchFindings` followed, when it returns no error, by `LaterDeleteReferences` over the pending cards as both the deleting and the later set, so dispatch keeps the check.
`validateFormatAfter` calls `dispatchFindings` instead of `ValidateDispatch`, so with a done card the plan gate reports each reference once, from `planGatePass`'s typed form, never also the scan-only duplicate; its exported doc says the check set it runs.

`LaterDeleteReferences` becomes scan-only (Decision 4): it resolves its targets and builds its regions as today (its `glyph-rejected` arm for an unreadable Delete or Edit answer stays), takes its subjects from `coverageSubjects` over a view of the plan holding only the deleting cards, keeping deleted members alone, and calls `memberReferences` with a nil loader over the region files that end in `.go`, pass `ignoredByGoTool` and exist on disk, never walking the tree for candidate files (coverageSubjects' go.mod directory walk for the import path still runs once); then `deleteOrderFindings` over the regions of the later cards, plus `laterImportFindings` for the package self glyph Deletes.
So a package-level reference stays blocking and a method match there is informational.
`deletedSymbol` and the regex construction go.
`Index.LaterDeleteReferences`'s comment says the findings are blocking for a resolved reference and informational for a name-only method match.
The file comments of `deleteorder.go`, `callercoverage.go`, `plangate.go` and `typesload.go` say the computation is shared; the load's log lines and `typesLoadSite` keep their wording.
`loom-plan-spec.md`'s row 32 is rewritten: a reference is found through `caller-uncovered`'s computation (row 35), typed where the load answers and by the import-path scan elsewhere, never textually, in `.go` files outside the directories the go tool ignores; a resolved reference inside a later card's `Edit:` region is blocking, one finding per line, and a method matched by name alone is one informational finding per deleted method naming every file and line; a package self glyph Delete keeps its quoted import-path match, searched only in `.go` regions; it is emitted by the plan-gate pass at `ValidateFormat`, `Validate`, `ValidateRework` and `ValidateFormatAfter` over its pending cards, and scan-only at `ValidateDispatch`, so a method match there is informational; the bound of Decision 2 and Decision 3 is stated.
`doc.go`'s `delete-before-reference` bullet says the same: blocking for a resolved reference and informational for a name-only method match, computed by `callerCoverageFindings` through `planGatePass` at the plan gates and by `LaterDeleteReferences` scan-only at dispatch and for Webster.

What this lets through, and its bound (Decision 2): a method call in a later region is only informational where no type information exists — a failed or timed-out load, a file with a parse error, a nested module, an untyped identifier, or dispatch, which runs no load — and the later card's verify build catches a real call there; a prose mention in a non-Go target, and a reference in a directory the go tool ignores, are no longer flagged, and neither breaks a build.

Tests:
A delete-before-reference expectation below is a severity and a location: `<file>:<line>` for a blocking finding, `<file> at line <n>` for an informational one; a row passes when its findings and its expectations pair one to one on severity, with each finding's detail holding its location.
- `TestLaterDeleteReferences` (scan-only) takes `wantRefs` in that shape; its loop keeps requiring the later card's ID and the deleted ref in every detail, which the informational detail carries.
  The row "a method target matches as a selector" now wants `informational sub/d.go at line 3`.
  New rows:
  a `sub#` Delete with a later `docs/notes.md#` Edit whose text quotes `"x/sub"` yields nothing (a `.md` region holds no Go identifier, so only the import-path match could reach one);
  a later Edit of `sub/g.go#`, whose only mentions of `Foo` are a comment and a string literal, yields nothing;
  a later Edit of `other/testdata/x.go#`, which imports `x/sub` and calls `sub.Foo()`, yields nothing for a `sub#Foo` Delete and nothing for a `sub#` Delete;
  a later Edit of `sub/a.go#`, which holds `Foo`'s own declaration, yields nothing;
  a later Edit of a `.go` file absent from disk yields nothing and no error, for a `sub#Foo` and for a `sub#` Delete.
  The extra files go in the rows' `extraFiles`.
- `TestGlyphChain_CallerCoverage` (untagged, failing loader) turns `wantDeleteOrder` into a sorted list in that shape, read from each finding's detail, and `coverageFiles` gains `docs/notes.md`.
  The two existing rows want `blocking callers/callers.go:6` and the `partuse_a.go` line their fixture calls `Target` on.
  New rows:
  deleting `callees#Thing.Method` with a later card editing `callers/callers.go` wants `informational callers/callers.go at line 9`, no blocking delete-before-reference finding and no caller-uncovered finding;
  `coveredTargetCard(covers)`, whose own Edit covers `callers#UseTarget`, followed by a card editing `callers#UseTarget` wants `blocking callers/callers.go:6` and no caller-uncovered finding, so a covered line inside a later region is delete-before-reference's;
  deleting `other#` with a later card editing `callees/callees.go` and `docs/notes.md`, where the note quotes `"example.com/glyphchain/other"`, wants only `blocking callees/callees.go:5`, pinning that `planGatePass` runs the import-path match over `.go` regions alone.
- `TestGlyphChain_CallerCoverageTyped` (integration, real load) wraps every row's loader in `countingTypesLoader`, which counts its calls and forwards them and which, with its load method, lives in glyphchain_integration_test.go beside the test, and asserts at most one load per row, exactly one on the rows below that report both checks.
  It gains `wantDeleteOrder` in that shape, `done` (the number of leading cards passed as done, run through `validateFormatAfter` instead of `planGatePass`) and `wantDispatchDeleteOrder` (the same shape, from `ValidateDispatch` over the same plan, which takes no loader).
  `coverageFiles` gains `callers/uses.go`, `callers/unrelated.go`, `callers/switch.go` and `callees/runner.go`.
  New rows, each with its caller-uncovered `want`:
  the motivating shape — card 1 edits `callers/switch.go`, card 2 deletes `callees#Runner.Run` and `callees#Runner.Call` (declared in a row file `callees/runner.go`), card 3 edits `callers/unrelated.go#`, which calls `(*exec.Cmd).Run`, a local type's `Run` and `reflect.Value.Call` and names `Run` and `Call` in a comment — wants no delete-before-reference finding and no caller-uncovered finding;
  card 1 deletes `callees#Thing.Method` and card 2 edits `callers/uses.go#`, which calls it on a `callees.Thing`, a `*callees.Thing`, a type embedding `callees.Thing` and as a method value on four lines — wants one blocking finding per line, `want` `blocking callers/callers.go`, one load, and from `ValidateDispatch` one informational finding naming each of the four lines;
  the same Delete with card 2 editing `broken/broken.go#`, whose call on line 7 follows a parse error on line 6, wants `informational broken/broken.go at line 7`, no blocking delete-before-reference finding, and `want` `blocking callers/callers.go`;
  card 1 edits `callees/local.go` and is done, card 2 deletes `callees#Thing.Method`, card 3 edits `callers/callers.go` — wants exactly `blocking callers/callers.go:9` and no caller-uncovered finding;
  card 1 deletes `callees#Target`, card 2 edits `callers#UseTarget` and `callees#CallsTarget` — wants `blocking callers/callers.go:6` and `blocking callees/local.go:6`, and `want` `blocking callees/callees_external_test.go`.
That a plan-gate pass runs one load is pinned by the count; that dispatch runs none holds by construction, since `ValidateDispatch` and `LaterDeleteReferences` take no loader and pass nil, and is observed as the scan-only verdict the typed rows' `wantDispatchDeleteOrder` asserts.

**ImpactSummary:** `planglyph` internals only, plus one interface comment in `planindex`; `LaterDeleteReferences`' signature is unchanged, so `websterengine` and `webstercli` compile untouched, and their package-level fixtures stay blocking under the scan.

**Commit:** 1: delete-before-reference shares caller-uncovered's reference computation
