---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 (lines 100-109) and D6 (line 113)
    summary: envkit.NilSeams walks every nested struct of shedrecipe.Env, but D5 never says which structs a NilSeams test marks and its scan checks only that some marker exists, so a card adding a field to a nested Deps such as landingshed.Deps is never held by D6
    fix: In D5, state that a test's markers name every struct its reflection walks, nested struct fields included, and that a test reaching envkit.NilSeams marks shedrecipe#Env and each struct type Env nests by value (recursively); make the tier0 scan derive that nested set from the Env type and refuse a NilSeams-reaching test whose markers miss one of them. Replace the rationale's claim that the scan keeps the marker exact with what the scan checks. Add an acceptance clause to the D5 bullet that a NilSeams test missing a nested struct's marker fails the scan, and one to the D6 bullet that a card adding a field to a nested Deps struct without its seam test fails with seam-test-unlisted.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4 (lines 79, 88)
    summary: D4 resolves pkg.Member.Sub as the glyph <dir>#Member.Sub, but quarry gives Go struct fields no glyph, so a cited real field such as shedrecipe.Env.PlanIndex yields a false citation-symbol-missing whose way forward cannot be met
    fix: In D4's second shape, resolve pkg.Member.Sub by its pkg.Member prefix only (the glyph <dir>#Member), with Sub untested, so a field or a method citation passes when its type exists; add to D4's Bound that a field or method after the first member is not tested, and add to the D4 acceptance bullet that an existing qualified struct field such as shedrecipe.Env.PlanIndex passes.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 Bound (lines 122-123)
    summary: D6's later-card cover lets a card that breaks a seam test pass when any later card edits that test for any reason, and D6's Bound states neither this nor what catches the broken test between the two cards
    fix: Append to D6's Bound one sentence stating that a seam test a later card names under Edit or Extends passes the earlier card even when the later card edits it for another reason and the test stays broken between the two cards, and that the earlier card's gate test run and review catch that.
  - id: F4
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 (lines 81-86) and D8 (line 146)
    summary: D4's resolving method states the outcome only for found, build-constraint-partitioned ambiguous and an outage; an unpartitioned ambiguous answer and a pre-resolution rejection have no stated discussion finding and no D8 classification
    fix: In D4, after the partitioned-ambiguity sentence, state that an unpartitioned ambiguous answer is a citation-symbol-ambiguous finding naming the candidates' files and a rejected answer a citation-symbol-missing finding, each with the way forward D4 already gives; in D8, state that a target whose answer is neither found nor partitioned counts as non-test code.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 (lines 146-151)
    summary: D8 lets an unresolved target count as test code only as a Test-prefixed Create member glyph, so a Rename's new side such as internal/x#TestNew, or a non-Test helper created by member glyph in a _test.go file, counts as non-test code and raises doc-go-unlisted on a test-only card, against D8's Bound
    fix: In D8's classification sentence, extend the unresolved case to any Create member glyph or Rename or Move new side whose other side, or whose Create group's file target, is test code, and to a member whose name begins Test, Benchmark, Fuzz or Example; add a clause to the D8 acceptance bullet that a card renaming a test function passes.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D6 (lines 116-119)
    summary: D6 counts a test's file named under Edit or Extends, by the card or a later card, as cover, but planparser.CheckEditTestFile refuses a _test.go file self glyph under Edit or Extends, so that route only trades one finding for edit-test-file
    fix: In D6, drop "or its file" from the later-card cover and limit the own-group file match to Uses, Delete, Rename and Move; make the seam-test-unlisted way forward say to list the test under Edit by its member glyph.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 trigger and Bound (lines 104, 107)
    summary: the scan follows only a same-package _test.go helper or envkit.NilSeams, so a test reaching reflection through a same-package non-test helper (internal/state/keep.go, internal/boardengine/task.go) is unseen, while the Bound admits only cross-package helpers
    fix: Change D5's Bound to say reflection reached through any non-test helper, in the same package or another, is not seen, except envkit.NilSeams, which the scan names.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 (line 113) and D9 (line 159)
    summary: D6 holds a card that names the struct's declaring file, which needs a quarry answer for the file, but D9 hands planContextFindings the index only for D8's classification and the seam-marker reader yields glyphs alone
    fix: In D9, change "for D8's classification" to "for D6's declaring-file match and D8's classification".
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 (line 71) and D4 (line 79)
    summary: D3 exempts a token holding a < as a placeholder, but D4's pkg.Member shape does not exclude one, so a negative-scenario citation such as loomshed.<Gone> reaches quarry and is rejected
    fix: In D4's second bullet, add that the token holds no whitespace and no <, as the glyph bullet already says.
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 glyph shape (line 78)
    summary: a backticked markdown anchor whose unit is a .md path, such as docs/overview.md#Modules, meets D4's glyph shape and is resolved, giving a false citation-symbol-missing on a valid citation
    fix: In D4's glyph bullet, add that the unit's last segment, when it holds a dot, ends in a source extension quarry has an alphabet for, so a .md or other non-source anchor is not a glyph; add a clause to the D4 acceptance bullet that a markdown anchor is untested.
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:goal-scope
    location: _lym/discussion/decision-record.md Auto-mode assumptions (lines 186-191)
    summary: D7 relaxes the entry's exactly one tier tag to also allow a || disjunction and untagged, but unlike the other readings of the entry it is not listed under Auto-mode assumptions
    fix: Add one bullet to Auto-mode assumptions reading that a new test file may state a || disjunction for a shared helper or untagged for a support-only file, wider than the entry's exactly one tier tag (D7).
---

### [MEDIUM] D5's marker set misses the structs NilSeams walks

**Location:** `_lym/discussion/decision-record.md` D5 and D6

**Issue:** `envkit.NilSeams` calls `walkNil`, which recurses on every `reflect.Struct` field (`internal/testkit/envkit/envkit.go:527-544`).
`shedrecipe.Env` nests `websterengine.RunDeps`, `landingshed.Deps`, `pawlengine.Deps`, `loomshed.PRReworkDeps` and `loomshed.DarnDeps` by value (`internal/shedrecipe/recipe.go:187-276`).
D5 asks for "one line per struct" a test reflects over, but its scan only refuses a test with no marker or a marker naming a non-struct.
So `//lym:seam internal/shedrecipe#Env` alone passes on `internal/loomcli/wiring_seams_test.go`.
D6 then holds only cards that name `Env`, so a card adding a func field to `landingshed.Deps` breaks the wire-seams tests with no `seam-test-unlisted` finding.
That is the entry's own `Deps`/`Env` case.
D5's rationale claims the scan keeps the marker set exact, but it does not.
Lenses interactions, walkthrough and bound-breaker and the handler found this independently.

### [MEDIUM] D4 resolves struct fields that quarry never glyphs

**Location:** `_lym/discussion/decision-record.md` D4

**Issue:** D4 turns `pkg.Member.Sub` into `<dir>#Member.Sub`.
Quarry's `docs/glyph.md` lists Go struct fields among the declarations that get no glyph.
So `shedrecipe.Env.PlanIndex`, a real field at `internal/shedrecipe/recipe.go:279`, resolves not_found and is refused as `citation-symbol-missing`.
That finding's way forward, correcting the token as `lym quarry glyphs` lists the members, cannot be met, because no listing holds a field.

### [MEDIUM] D6's later-card cover has no stated bound

**Location:** `_lym/discussion/decision-record.md` D6 Bound

**Issue:** A later card that names the seam test under Edit or Extends covers an earlier card that breaks it, whatever the later edit is for.
The test then stays broken from the earlier card until the later one.
The rubric's attack-surface rule wants the Decision to say what this hatch lets through and what bounds it.
D8's Bound states its matching later-card hatch, but D6's Bound covers only the Uses listing.
Bound-breaker and the handler found this; the focus directive asked for it.

### [LOW] Unpartitioned ambiguous and rejected answers have no outcome

**Location:** `_lym/discussion/decision-record.md` D4 and D8

**Issue:** The status policy D4 lifts out of `statusFindings` maps an unpartitioned ambiguous answer to `glyph-ambiguous` and a status-less answer to `glyph-rejected` (`internal/planglyph/resolve.go:94-153`).
Neither has a discussion-side finding name.
D8 does not say how it classifies either answer.
Interactions, bound-breaker and the handler raised this.

### [LOW] D8 misclassifies a renamed or helper test member

**Location:** `_lym/discussion/decision-record.md` D8

**Issue:** Rename pairs project both endpoints into the targets (`internal/planglyph/resolve.go:13-17`).
A card renaming `internal/x#TestOld` to `internal/x#TestNew` has a new side that does not resolve and is not a Create, so D8 counts it as non-test code.
The same happens to a non-`Test` helper created by member glyph.
Each raises `doc-go-unlisted` on a test-only card, which D8's Bound says is not checked.
Walkthrough, bound-breaker and the handler raised this.

### [LOW] D6's cover by test file under Edit collides with edit-test-file

**Location:** `_lym/discussion/decision-record.md` D6

**Issue:** `planparser.CheckEditTestFile` refuses any `_test.go` file self glyph under Edit or Extends in a card that is not done (`internal/planparser/edittestfile.go:13-51`).
D6 accepts a test named "by its file" as cover, including by a later card under Edit or Extends.
That route swaps `seam-test-unlisted` for `edit-test-file`.

### [LOW] D5's Bound omits same-package non-test helpers

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** The trigger follows only a same-package `_test.go` helper or `envkit.NilSeams`.
Package `state`'s tests can reach `reflect.VisibleFields` through `internal/state/keep.go:176`, and package `boardengine`'s tests can reach `NumField` through `internal/boardengine/task.go:68`, both unseen.
D5's Bound names only helpers in another package.

### [LOW] D9 scopes the index to D8 though D6 needs it too

**Location:** `_lym/discussion/decision-record.md` D6 and D9

**Issue:** D6 holds a card that names the struct's declaring file, and finding that file needs the index, since D4's method reports declaring files.
D9 says `planContextFindings` gains the index for D8's classification only.

### [LOW] D4's pkg.Member shape admits a placeholder token

**Location:** `_lym/discussion/decision-record.md` D3 and D4

**Issue:** D3 and D10 make `<placeholder>` the way to cite something that must not exist.
D4 excludes `<` only from the glyph shape.
A citation such as `loomshed.<Gone>` therefore reaches quarry, which rejects it before resolution.

### [LOW] Markdown anchors read as glyphs

**Location:** `_lym/discussion/decision-record.md` D4

**Issue:** A backticked anchor whose unit is a `.md` path and whose fragment parses under the glyph grammar meets every condition in D4's glyph shape.
D3 skips it for the `#`.
D4 resolves it against an alphabet quarry does not have, which yields `citation-symbol-missing` on a valid citation, and the only way past is a false ` (new)` mark.

### [NIT] D7's relaxation is missing from Auto-mode assumptions

**Location:** `_lym/discussion/decision-record.md` Auto-mode assumptions

**Issue:** The entry says "exactly one tier tag".
D7 also accepts a `||` disjunction or `untagged`, and gives its reason in its own text.
Every other reading of the entry is listed under Auto-mode assumptions, but this one is not.

## Focus verdicts

- D4 resolution method: the Decision names the new `planindex.Index` method, its inputs (a list of glyphs and a worktree root), its outputs (resolution and declaring files) and the outage outcome. The unpartitioned ambiguous and rejected outcomes remain open (F4).
- D5 scan trigger: every test the directive names is caught, namely the `NumField` calls on a `reflect.Type` in the three config and mergevocab tests, `reflect.VisibleFields` as a package function, and unqualified `NilSeams` inside `envkit`; not a defect. The marker contents behind the trigger are the defect (F1).
- D8 classification: the rule is stated, and it agrees with the Bound and the acceptance bullet for an Edit of a test function by member glyph. It breaks for Rename new sides and non-`Test` helpers (F5).
- Escape hatches: D8's Uses, Extends and later-card hatches are bounded, and so is D6's Uses hatch. D6's later-card hatch is not (F3).

## Rejected

- None of the lens findings was rejected. Existing-paths reported no finding.
