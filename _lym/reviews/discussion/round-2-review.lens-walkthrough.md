```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Decisions walked:**
- D1, this record's own `## Decisions` H3s (all `### D<n> — …`, ids unique): purpose met.
- D2, this record's acceptance bullets (D1 to D11 each spelled at least once; "D1 is not matched by D10 alone" whole-word): purpose met.
- D3, this record's paths `docs/research/review-setup-audit-2026-10-28/report.md`, `cmd/lym/tiertags_test.go` and `pattern/PATTERN-test-speed.md`, plus the `.scratch/` and `internal/...` exemptions: purpose met.
- D4, case A: `planindex.ErrQuarryUnavailable` gives `internal/planindex#ErrQuarryUnavailable`, which is found: purpose met.
- D4, case B: `pkg.Member.Sub` where `Sub` is a struct field, such as `shedrecipe.Env.PlanIndex`: purpose missed (F1).
- D5, `internal/testkit/envkit/envkit_test.go` `TestFullEnv_PathFieldsAbsoluteUnderOneRoot` (`v.NumField()` on a `reflect.Value`) and `TestNilSeams_*` (`NilSeams` unqualified inside envkit): the scan catches both, purpose met. The marker content for `NilSeams` callers is the problem; see F2.
- D5 with D6, a card adding a func field to `internal/landingshed#Deps`, which `shedrecipe.Env` nests by value as `Landing`: purpose missed (F2).
- D6, a card editing `internal/shedrecipe#Env` with the marked test's file under Uses: passes, purpose met. The "later card names its file under Edit or Extends" route is refused elsewhere (F4).
- D7, a card that creates `internal/x/foo_test.go` with an Intent line naming `//go:build tier0`: purpose met. Hub `gate.yaml` tier keys `tier0` to `tier3` equal the `-tags` values.
- D8, case A: a card editing only `internal/loomshed#TestNewDiscussionGate`, which resolves declared only in `_test.go`, counts as test code: purpose met.
- D8, case B: a Rename pair `internal/x#TestOld -> internal/x#TestNew`: purpose missed (F3).
- D9, `ValidatePlan` and `ValidateReworkPlan` both hold `index` and call `planContextFindings`: purpose met.
- D10 and D11: no mechanism to run past the text edits; not walked.

**Findings:**

**F1 — MEDIUM, design, D4 (decision-record.md:79).**
D4 resolves `pkg.Member.Sub` as the glyph `<dir>#Member.Sub`, but Go struct fields are not glyphs in quarry.
So a cited real field such as `shedrecipe.Env.PlanIndex` is `not_found` and becomes a false `citation-symbol-missing`.
Its way forward ("correct it as `lym quarry glyphs` lists") cannot be met, because no listing holds the field.
The Bound in decision-record.md:88 does not name fields.
- Evidence: quarry v0.4.0 `docs/glyph.md:216` lists Go "struct fields" among the non-glyph declarations.
- Evidence: `internal/shedrecipe/recipe.go:279` shows `PlanIndex` is a field of `Env`.

**F2 — MEDIUM, design, D5 and D6 (decision-record.md:100, 104, 113).**
`envkit.NilSeams` recurses into every nested struct field of `shedrecipe.Env`.
D5 marks "one line per struct" the test reflects over, and its scan checks only that a marker exists and names a struct, not which structs.
So a `NilSeams` caller is marked `internal/shedrecipe#Env`.
A card adding a func field to `internal/landingshed#Deps`, nested as `Env.Landing`, names neither `Env`, a member of it, its file nor its package, so D6 does not hold it.
No `seam-test-unlisted` fires, yet `wiring_seams_test.go` breaks.
This is the board entry's own "field to a shared `Deps`" case.
D5's rationale ("a marker kept complete by a scan is exact") does not hold for nested structs.
- Evidence: `internal/testkit/envkit/envkit.go:543-544` (`case reflect.Struct: walkNil(fv, path, nils)`).
- Evidence: `internal/shedrecipe/recipe.go:187, 196, 223, 266, 276` (`WebsterDeps websterengine.RunDeps`, `Landing landingshed.Deps`, `Pawl pawlengine.Deps`, `Rework loomshed.PRReworkDeps`, `Darn loomshed.DarnDeps`).
- Evidence: `internal/landingshed/deps.go:35` (`type Deps struct`).
- Evidence: `internal/loomcli/wiring_seams_test.go:116`.

**F3 — LOW, consistency, D8 (decision-record.md:146-151).**
D8 grants an unresolved target test status only when it is a Create member glyph with a `Test…`/`Benchmark…`/`Fuzz…`/`Example…` name.
A Rename pair's new side, such as `internal/x#TestNew`, does not resolve and is not Create, so it counts as non-test code.
A card that only renames a test function therefore gets `doc-go-unlisted`.
That contradicts D8's Bound, "a test-only card … not checked".
- Evidence: `internal/planglyph/resolve.go:13-17`, where both Pairs endpoints are projected into the card's targets.

**F4 — LOW, consistency, D6 (decision-record.md:116-119).**
D6 counts a card or later card that names the seam test "by its file" under Edit or Extends as coverage.
But `planparser.CheckEditTestFile` refuses any `_test.go` file self glyph under Edit or Extends in a card not done.
So that route only trades `seam-test-unlisted` for `edit-test-file`, and only member glyphs (or Uses) can cover.
D6 neither notes this nor steers the way forward to a member glyph.
- Evidence: `internal/planparser/edittestfile.go:13-17, 35-51`.

**Tool calls:**
1. `internal/landingshed/deps.go` `type Deps` and `internal/planparser/edittestfile.go` `CheckEditTestFile`: the manifest lacked the nested `Deps` type and the existing Edit-of-test-file refusal that D6's cover meets.
2. quarry `docs/glyph.md` field lines, and the location of the hub `gate.yaml`: the manifest lacked whether Go struct fields are glyphs, and where the tier config lives.
3. The tier keys and `-tags` in `_board/_lym/config/gate.yaml`: the manifest lacked the hub's tier keys that D7's assumption rests on.
