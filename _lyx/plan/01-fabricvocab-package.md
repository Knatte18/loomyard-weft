# Card 1 — fabricvocab-package

**Create:**
- `internal/fabricvocab/fabricvocab.go`
- `internal/fabricvocab/doc.go`
- `internal/fabricvocab/leaf_enforcement_test.go`
- `plan:internal/fabricvocab#ConfigsyncOwnerDir` -> `const ConfigsyncOwnerDir = "internal/configsync"`
- `plan:internal/fabricvocab#owners` -> `var owners map[string]bool`
- `plan:internal/fabricvocab#docRoots` -> `var docRoots map[string]bool`
- `plan:internal/fabricvocab#hostGeometryIdentifiers` -> `var hostGeometryIdentifiers map[string]bool`
- `plan:internal/fabricvocab#sideTokens` -> `var sideTokens []string`
- `plan:internal/fabricvocab#packageNamePattern` -> `var packageNamePattern *regexp.Regexp`
- `plan:internal/fabricvocab#SideTokens` -> `func SideTokens() []string`
- `plan:internal/fabricvocab#BareToken` -> `func BareToken(s string) bool`
- `plan:internal/fabricvocab#BareTokenIn` -> `func BareTokenIn(s string, stripPackageName bool) bool`
- `plan:internal/fabricvocab#IsHostGeometryIdentifier` -> `func IsHostGeometryIdentifier(name string) bool`
- `plan:internal/fabricvocab#IsOwner` -> `func IsOwner(dir string) bool`
- `plan:internal/fabricvocab#SkipsBareCheck` -> `func SkipsBareCheck(dir string) bool`
- `plan:internal/fabricvocab#FailsBareCheck` -> `func FailsBareCheck(dir string, bareIdent, bareLiteralOrComment bool) bool`
- `plan:internal/fabricvocab#IsDocRoot` -> `func IsDocRoot(first string) bool`
- `plan:internal/fabricvocab#allowedImports` -> `var allowedImports []string`
- `plan:internal/fabricvocab#TestLeafInvariant_AllowlistOnly` -> `func TestLeafInvariant_AllowlistOnly(t *testing.T)`

**Edit:**
- `internal/lyxcwd#fabricVocabularyFailures`
- `internal/lyxcwd#fabricVocabularyHits`
- `internal/lyxcwd#TestEnforcement_FabricVocabulary`
- `pattern/PATTERN-fabric-vocabulary.md`
- `pattern/PATTERN-leaf-packages.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/lyxcwd#configsyncOwnerDir`
- `internal/lyxcwd#fabricVocabularyOwners`
- `internal/lyxcwd#weftnamePackagePattern`
- `internal/lyxcwd#vocabScanDocRoots`
- `internal/lyxcwd#hostGeometryIdentifiers`
- `internal/lyxcwd#bareVocabularyToken`
- `internal/lyxcwd#bareVocabularyTokenIn`
- `internal/lyxcwd#shouldSkipBareVocabularyCheck`
- `internal/lyxcwd#failsBareVocabularyCheck`

**Uses:**
- `internal/lyxcwd#vocabScanAllowlist`
- `internal/lyxcwd#vocabScanSkippedRoots`
- `internal/lyxcwd#importsWeftname`

**Intent:**
D10: move the fabric-vocabulary predicates out of `internal/lyxcwd/enforcement_test.go` into a new production leaf package, `internal/fabricvocab`, so the D9 plan check (card 2) and `TestEnforcement_FabricVocabulary` call one source and cannot drift.

`internal/fabricvocab/fabricvocab.go` (package `fabricvocab`, standard library only) holds the predicates and their data, moved verbatim in behavior:
- `sideTokens` holds the two fabric side tokens, and `SideTokens` returns a fresh copy, so a test outside the owner set can build a side-token name without spelling one.
- `BareToken` and `BareTokenIn` are today's `bareVocabularyToken` and `bareVocabularyTokenIn`: a case-insensitive substring match on either side token, with `BareTokenIn`'s second argument stripping the side-named name package's spelling (today's `weftnamePackagePattern`, now `packageNamePattern`) first.
  The exported parameter is named `stripPackageName`, so no exported spelling a non-owner caller sees carries a side token (`PATTERN-fabric-vocabulary`: "A non-owner file carries no side-named spelling, owner-set exports included").
- `IsHostGeometryIdentifier` is today's `hostGeometryIdentifiers[strings.ToLower(name)]` lookup over the moved `hostGeometryIdentifiers` map.
- `owners` is today's `fabricVocabularyOwners` plus `internal/fabricvocab` itself, since the package spells the tokens it bans; `IsOwner` reads it.
  `ConfigsyncOwnerDir`, `SkipsBareCheck` and `FailsBareCheck` are today's `configsyncOwnerDir`, `shouldSkipBareVocabularyCheck` and `failsBareVocabularyCheck`.
- `docRoots` is today's `vocabScanDocRoots`; `IsDocRoot` reads it by a path's first segment.

The host-phrase list, `fabricSenseHostPhrase`, the allowlist, the skipped roots, the walk, `fabricVocabularyHits` and the import rule stay in the test.
Keep the package's comments and string literals free of every policed host phrase: the host-phrase rule scans non-test Go under `internal/`, owner set included.

`internal/lyxcwd/enforcement_test.go` deletes the moved declarations and calls the package: `fabricVocabularyFailures` uses `fabricvocab.IsDocRoot`, `fabricvocab.SkipsBareCheck`, `fabricvocab.FailsBareCheck`, `fabricvocab.IsOwner` and `fabricvocab.BareToken`; `fabricVocabularyHits` uses `fabricvocab.BareTokenIn` and `fabricvocab.IsHostGeometryIdentifier`; `TestEnforcement_FabricVocabulary`'s predicate subtests call the moved names and `fabricvocab.ConfigsyncOwnerDir`, each subtest keeping its fixture and assertion, so the scan's behavior is unchanged (acceptance: "`TestEnforcement_FabricVocabulary` and its fixtures pass unchanged in behavior, calling the moved predicates").
`internal/lyxcwd/vocabscan_test.go` stays untouched and still passes.

`internal/fabricvocab/doc.go` carries the package godoc: what the predicates decide, that `TestEnforcement_FabricVocabulary` and `planparser`'s `fabric-vocabulary-name` check (card 2) are its callers by role, why it is a production package (production code cannot import a test file or a `testkit` kit), and that it imports only the standard library.
`internal/fabricvocab/leaf_enforcement_test.go` follows `internal/discussionparser/leaf_enforcement_test.go`: `TestLeafInvariant_AllowlistOnly` calls `scankit.AssertImportAllowlist(t, "internal/fabricvocab", allowedImports...)` with an empty `allowedImports`.

`pattern/PATTERN-fabric-vocabulary.md` adds `fabricvocab` to the owner-set bullet and names `internal/fabricvocab` as the home of the predicates the scan and the plan check share.
`pattern/PATTERN-leaf-packages.md` gains an `internal/fabricvocab` section: "Imports only the standard library." plus one line that it holds only the vocabulary predicates and their data, since the owner-set row exempts the whole package from the bare-token rule.
`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/fabricvocab: 1`.

**ImpactSummary:** test-only callers in `internal/lyxcwd` switch to a new stdlib-only package; no production behavior changes.

**Commit:** `1: Move the fabric-vocabulary predicates into internal/fabricvocab`
