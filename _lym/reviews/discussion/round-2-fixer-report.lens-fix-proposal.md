```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer applied as proposed: F1, F2, F3, F4, F5, F6, F7, F8, F9 and F11. F10 was fixed differently and is not in my scope.

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F8, F9, F11.

1. **MEDIUM.** D8, the new unresolved-Create clause: "a Create member glyph in a card whose Create group names a `_test.go` file target".
   Summary: a card whose Create group makes a new `_test.go` file and also a new production function by member glyph has that production function counted as test code. The card then looks test-only and its `doc.go` is never checked.
   Evidence: take a card with `**Create:**` holding `internal/x/foo_test.go` and `internal/x#NewParser`.
   - Under the new clause, `internal/x#NewParser` does not resolve yet, and its card's Create group names a `_test.go` file target, so it is test code.
   - Every target of the card is then test code, and D8's Bound says "a test-only card … not checked".
   - So a card adding production code passes with no `doc-go-unlisted`. D8's Bound does not mention this hole.
   - The review's proposal ("whose Create group's file target … is test code") was meant to place a helper in a test file. The clause as written tests the card, not the file the member is declared in.

2. **MEDIUM.** D4, the new sentence: "An ambiguous answer not split by build constraint is a `citation-symbol-ambiguous` finding naming the candidates' files … each with the way forward above".
   Summary: the way forward reused for this answer, "spell the symbol as a glyph", cannot be met when the token already is a glyph, or when its glyph is itself ambiguous.
   Evidence:
   - Quarry's `docs/glyph.md:265` lists `ambiguous` for Go build-tag duplicates that are not mutually exclusive, and for a match in two alphabets.
   - Spelling `pkg.Member` as `<dir>#Member` gives the same glyph and the same ambiguous answer. A glyph-shaped token has no more precise spelling.
   - The only ways past are a false ` (new)` or ` (removed)` mark or an artificial `<` placeholder, the same dead end F2 removed for struct fields.

3. **LOW.** D5, the new sentence: "a test reaching `envkit.NilSeams` marks `internal/shedrecipe#Env` and each struct type `Env` nests by value, recursively".
   Summary: the nested set the scan would derive includes structs that `walkNil` never walks, so the scan demands markers the test's reflection does not need.
   Evidence:
   - `walkNil` skips unexported fields (`internal/testkit/envkit/envkit.go:531-533`) and paths listed in `nilLegal` (`envkit.go:538-540`).
   - The fixed text says "each struct type `Env` nests by value" with neither exclusion. A struct reached only through an unexported or `nilLegal` field would be demanded as a marker.
   - D6 would then hold cards to it, so a card changing that struct gets a `seam-test-unlisted` finding about a test that can never break from the change.

4. **LOW.** D5 and D6 together, with the F1 change.
   Summary: the derived nested set is large, and the record does not state the plan-time cost.
   Evidence:
   - The set includes `websterengine.RunDeps` and the structs it nests by value: `websterengine.Config`, `Geometry`, `shuttleengine.Config`, `shuttleengine.GateSpec`. Also `landingshed.Deps`, `landingshed.Config`, `ParentLockDeps`, `pawlengine.Paths`, `driftengine.Paths`, `battenshed.RowNoticeDeps`, `battenshed.PrimeLock`, and others.
   - `NilSeams` is called from six test files: `internal/loomcli/wiring_seams_test.go`, `wiring_darn_test.go`, `wiring_beam_test.go`, `internal/shedrecipe/entries_batten_test.go`, `internal/battencli/wire_seams_test.go` and `internal/testkit/envkit/envkit_test.go`.
   - Under D6's declaring-file and package-self-glyph triggers, a card editing any of those structs' declaring files, such as a websterengine config file, must cover every test that reaches `NilSeams` in those files.
   - Neither D6's Bound nor Open risks states this burden.

Tool calls:
- Listed `shedrecipe.Env`'s fields, the files calling `NilSeams` and the `nilLegal` set. My context showed only part of the `Env` field list and none of the callers.
- Grepped the struct types `Env` nests by value, one level down. My context did not show what the nested set derived under F1 would contain.
