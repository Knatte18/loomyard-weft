# Card 6 — seam-marker-scan

**Create:**
- `plan:internal/testkit/envkit#envSeamStructs` -> `func envSeamStructs() []string`
- `plan:internal/testkit/envkit#seamViolations` -> `func seamViolations(root string, dirs []string, envStructs []string) ([]string, error)`
- `plan:internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked` -> `func TestSeamMarkers_ReflectiveTestsAreMarked(t *testing.T)`

**Edit:**
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/websterengine_test#TestConfigTemplate`
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`
- `internal/fabricengine#TestMergeVocabulary_ResultAndOptionsFieldsAreSideFree`
- `internal/fabricengine_test#TestMergeState_Record`

**Uses:**
- `plan:internal/impactset#FindSeamTests`
- `plan:internal/impactset#SeamTest`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#NilSeams`
- `internal/testkit/envkit#walkNil`
- `internal/testkit/scankit#Root`
- `internal/testkit/scankit#Walk`

**Intent:**
D5's scan, and the markers on today's reflective tests.

`envSeamStructs` returns, sorted, the glyph of `shedrecipe.Env` and of each struct type `Env` nests by value, recursively, exactly as `walkNil` walks: an exported field whose dotted path `nilLegal` does not skip, of `reflect.Struct` kind, recursed into.
A struct type's glyph is its package path with the module prefix `github.com/Knatte18/loomyard/` removed, `#` and its name, and a struct from outside the module (such as `time.Time`) is walked into but not listed, since it has no worktree glyph.
It reads `reflect.Type` field sets, so it lives in a `_test.go` file of the package and its callers carry a marker.

`seamViolations` checks, per told package directory under `root`, its `_test.go` files, parsed with `go/parser`, and returns one line per violation, naming the file, the test and the way forward:

- a top-level `func Test…` carrying no `//lym:seam` marker (read through `impactset.FindSeamTests`) whose body calls a `.NumField()` selector, or `reflect.VisibleFields`, directly or through a function declared in a `_test.go` file of the same directory that does, followed transitively; way forward: "add a `//lym:seam <unit>#<Type>` line for each struct it reflects over";
- a test whose body reaches `envkit.NilSeams` the same ways (a selector `NilSeams` on the import name of `internal/testkit/envkit`, or, in package `envkit` itself, the bare identifier `NilSeams`) and whose markers miss any glyph of `envStructs`; the line names each missing glyph, with the same way forward;
- a marker whose glyph's unit directory (a trailing `_test` dropped) declares no `type <Name> struct` in any of its `.go` files, test files included; way forward: "correct the marker's glyph to name the struct the test reflects over".

A test reflecting over a struct declared outside the module marks it by its import path, such as `//lym:seam time#Time`: a marker whose unit's first path segment names no directory under `root` is taken as a struct outside the module and is not checked against a declaration, so both the unmarked and the marked state have a way to pass, and a marker under a module directory such as `internal/` is still checked.
For a test reflecting directly, the scan checks that a marker exists and, for a unit under a module directory, names a struct, not that the markers name every struct it walks.
Bound, as the PATTERN background states: reflection reached through a non-test helper, other than `envkit.NilSeams`, is not seen.

`TestSeamMarkers_ReflectiveTestsAreMarked`, tier0 in the existing test file of the package, carries `//lym:guard` (it reads every package's test files) and `//lym:seam internal/shedrecipe#Env` (it reaches `envSeamStructs`).
It runs `seamViolations` over every directory `scankit.Walk` yields a `_test.go` file in, under `scankit.Root`, and fails on any violation; that is the row proving today's tests carry their markers.
Its fixture rows, each a temp root with one package directory `p` declaring `type T struct{}`, `type Env struct{}` and `type Inner struct{}`, pass `envStructs` of `[]string{"p#Env", "p#Inner"}`: a test calling `typ.NumField()` unmarked is a violation and marked `//lym:seam p#T` is not; a test calling `reflect.VisibleFields` unmarked is a violation; a test calling a same-directory `_test.go` helper that calls `NumField` unmarked is a violation; a marker naming `p#F` where `F` is a func is a violation; a test calling `typ.NumField()` marked `//lym:seam time#Time` is not a violation, while one marked `//lym:seam p/missing#T` is; a test calling `envkit.NilSeams` unmarked is a violation, marked with `p#Env` alone is a violation naming `p#Inner`, and marked with both is not; a test in a file of package `envkit` calling the bare `NilSeams` marked with `p#Env` alone is a violation naming `p#Inner`; a test calling a same-directory `_test.go` helper that calls `envkit.NilSeams`, marked with `p#Env` alone, is a violation naming `p#Inner`.
Each violation line is asserted to hold the test's name and its way forward.
A row over the real type asserts that `envSeamStructs()` holds `internal/shedrecipe#Env` and `internal/landingshed#Deps`, which `Env` nests by value through `Landing`, and no glyph outside the module such as one for `time.Time`.

Markers added, each above the named test's func line in its existing comment block (stacked with its `//testtiming:keep` line where it has one):
`internal/websterengine#Config` on `TestConfigTemplate`;
`internal/loomengine#Config` on `TestConfigTemplate_ContainsEveryConfigYAMLTag`;
`internal/fabricengine#MergeResult` and `internal/fabricengine#MergeOptions` on `TestMergeVocabulary_ResultAndOptionsFieldsAreSideFree`, which reflects through `assertStructFieldsSideFree`;
`internal/fabricengine#MergeStateForTest` on `TestMergeState_Record`, which reflects through `assertNoZeroFields`;
`internal/landingshed#Deps` on `TestLandingDeps_EveryFieldPopulated`;
`internal/shedrecipe#Env` on `TestFullEnv_PathFieldsAbsoluteUnderOneRoot`;
and on each test reaching `NilSeams` (`TestFullEnv_FillsEverySeam`, `TestNilSeams_ReportsExactPaths`, `TestNilSeams_SkipsNilLegalPaths`, loomcli's `TestWire_EverySeamFilled`, `TestWire_DarnRecipe` and `TestWire_BeamRecipe`, and battencli's `TestWire_EverySeamFilled`) one marker per glyph `envSeamStructs` returns, in its sorted order.
No other line of these tests changes.

**ImpactSummary:** comment-only marker lines on the existing tests this card lists, plus one new repository scan in envkit; no production code changes.

**Commit:** `6: Refuse an unmarked reflective test and mark today's`
