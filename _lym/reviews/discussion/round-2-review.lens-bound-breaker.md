```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: attack the record's bounds, guards and claims about existing code with concrete counterexamples.

**Claims attacked**

- D4 "the gate gets [worktree root and index] from `shedrecipe.Env`, which already holds both": **held.** The loom wire sets `PlanIndex: websterGeom.Index` (`internal/loomcli/wiring.go:582`), and the validate wiring sets `c.env.PlanIndex` and `WorktreeRoot` (`internal/loomcli/wiring.go:296`, `:309`).
- D4 "a quarry outage is a returned error … never a pass": **held** as a stated contract (`ErrQuarryUnavailable`, `internal/planindex/planindex.go:16`).
- D4 "an ambiguous answer split by build constraint still resolves": **held**, but the unsplit ambiguous answer and the rejected answer have no stated outcome (F4).
- D4 glyph shape "a unit holding a `/` or a `.`": **broken** for markdown anchors (F5).
- D2 "D2 adds nothing on a record missing `## Acceptance criteria`" and "the darn record has the same required section": **held**, since darn runs the same `Validate` with `## Acceptance criteria` required (`internal/discussionparser/validate.go:2`, `:44`).
- D5 trigger against today's tests:
  - **held** for websterengine and loomengine `config_test.go`, fabricengine `mergevocab_test.go` and `mergestate_integration_test.go` (`typ.NumField`), and `envkit_test.go` (`v.NumField`, unqualified `NilSeams`).
  - `FieldByName` users are out of the trigger, and that is consistent with "field set".
- D5 Bound "reflection reached through a helper in another package is not seen, except `envkit.NilSeams`": **broken**, because the trigger also misses same-package non-test helpers (F3).
- D5 Rationale "a marker kept complete by a scan is exact": **broken** (F1).
- D6 Bound for the Uses listing: **held** (the card gate's test run and review).
- D6 later-card cover: **unbounded** (F2).
- D7 "tier tags are the hub's `gate.yaml` tier keys" and "with no `gate.yaml` the check is skipped": **held**.
  - The hub `gate.yaml` keys are `tier0` to `tier3`, and each command passes `-tags tierN`.
  - `validate-plan` already treats `gateslot.ErrConfigAbsent` as absent (`internal/loomcli/validate.go:262-271`).
- D8 classification against its acceptance bullet: **held** for an Edit of a test function by member glyph. It is **broken** for the new side of a Rename and for a test helper created by member glyph (F6).
- D9 "Both `PlanContext` builders": **held**, there are exactly two (`internal/loomcli/validate.go:272`, `internal/shedrecipe/entries_gate.go:127`).
- D9 "`--require-approved` loads no `PlanContext`": **held** (`internal/loomcli/validate.go:256`).
- D9 "`planContextFindings` gains the index its callers already hold": **held**, since `ValidatePlan` and `ValidateReworkPlan` both take `index` (`internal/loomshed/gates.go:237`, `:322`).

**Findings**

**F1. MEDIUM · design · D5 Rationale and D6.**
The scan does not keep the marker set complete, so the `Env`/`Deps` seam case the entry names goes unseen.
- `envkit.NilSeams` → `walkNil` recurses into every nested struct field of `shedrecipe.Env` (`internal/testkit/envkit/envkit.go:527-544`). Those fields include `Landing landingshed.Deps`, `Pawl pawlengine.Deps`, `Darn loomshed.DarnDeps` and `WebsterDeps websterengine.RunDeps` (`internal/shedrecipe/recipe.go:187-276`).
- The scan only refuses a reflective test with no marker at all, or a marker that names a non-struct.
- So `//lym:seam internal/shedrecipe#Env` alone satisfies it on `internal/loomcli/wiring_seams_test.go:116`.
- Counterexample: a card adds a func field to `landingshed.Deps`. D6 holds it to the markers of `landingshed#Deps`, finds none and reports nothing, yet the NilSeams wire-seam tests fail at the card gate.
- The record never says whether a test marks each nested struct its walk reaches, and no check could enforce it.

**F2. MEDIUM · design · D6 Bound.**
The later-card cover ("a later card names the test or its file under Edit or Extends") is an escape hatch with no stated bound.
- Counterexample: card 3 adds a field to a marked struct, and card 7 edits the seam test for an unrelated reason.
- D6 passes card 3, but between cards 3 and 7 the seam test is broken, and card 3's gate test run fails.
- D6's Bound covers the Uses listing only. D8's Bound states its later-card hatch; D6's does not.

**F3. LOW · consistency · D5 trigger and Bound.**
The trigger follows only a same-package `_test.go` helper or `envkit.NilSeams`. The Bound claims only cross-package helpers go unseen.
- Counterexample: package `state`'s own tests reaching `reflect.VisibleFields` through `internal/state/keep.go:176`, or package `boardengine`'s tests reaching `t.NumField()` at `internal/boardengine/task.go:68`.
- Each is a same-package non-test helper. The scan misses it, and the Bound does not admit it.

**F4. LOW · design · D4.**
Only partitioned ambiguity is given an outcome.
- `statusFindings` maps an unpartitioned ambiguous answer to `glyph-ambiguous` and a status-less answer to `glyph-rejected` (`internal/planglyph/resolve.go:94-153`).
- D4 says neither whether such a citation is resolved, `citation-symbol-missing`, or some other finding, nor what D8's classification reads for it.

**F5. LOW · design · D4 glyph shape.**
A backticked markdown anchor such as `pattern/PATTERN-test-speed.md#lym-guard` meets every glyph condition: no whitespace, no `<`, unit holds `/` and `.`.
- D3 skips it for the `#`, and D4 tries to resolve it.
- Quarry has no alphabet for `.md`, so it answers not_found or a rejection. That is a `citation-symbol-missing` finding on a valid citation.
- The only way out D4 offers is a false ` (new)` mark.

**F6. LOW · consistency · D8 test classification.**
Only a Create member glyph that does not resolve can be test code. Rename pairs project both endpoints into the targets (`internal/planglyph/resolve.go:14-46`).
- So a card renaming `internal/x#TestOld` to `internal/x#TestNew` has a new side that does not resolve and is not Create, which makes it non-test code. That raises `doc-go-unlisted` on a test-only card, contrary to D8's Bound "a test-only card … not checked".
- A test helper created by member glyph with a non-`Test` name hits the same misclassification.

**Tool calls**

1. `internal/loomcli/wiring.go:290-315`, the `PlanContext` construction sites, and the `--require-approved` handling in `validate.go`: to check the claims on Env fields and builders, which the manifest named but did not show.
2. Every `PlanIndex` use in loomcli, battencli and shedrecipe, plus `entries_gate.go:115-160`: to check the loom gate's Env carries an index.
3. Every `PlanIndex` site repo-wide, plus `wiring.go:570-590`: to confirm where the run Env's index is filled.
4. `discussionparser` required sections and the hub `_board/_lym/config/gate.yaml`: to check D2's darn claim and D7's tier-key claim.
5. `validate.go:252-281`, the `RefuseRaw` load: to check D7's "no `gate.yaml`" skip path.
