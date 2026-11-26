```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens on plan cards 1–14 (test seams, doc text the cards make false, card order).

**Per card, each invariant checked:**

- **Card 1**
  - New test tier: held. `TestKeyName` is tier0 in `configengine_test`.
  - Doc text: held. It extends `configengine/doc.go` and adds the `PATTERN.md` entry.
  - Order: held. It has no dependencies.
- **Card 2**
  - Seam: held. No reflective test walks `configengine.Spec`.
  - Doc text: held. It edits `doc.go`, `PATTERN.md` and `pattern/PATTERN-config-layers.md`, but see F2 for the background's "Loading config" rule line.
  - Order: held. No table is filled yet, so no load changes.
- **Card 3**
  - New test tier: held. `TestMoveKeys` is tier0.
  - Doc text: held. It extends `yamlengine/doc.go`.
  - Order: held.
- **Card 4**
  - Seam: held. It names `//lym:seam` for the reflective `Config` and `TapestryConfig` tests.
  - Doc text: held. `reviewbudget/doc.go` names the flat keys and is covered by the package glyph `internal/reviewbudget#`.
  - Order: broken in its text (F3).
- **Card 5**
  - Order: held. It needs card 2's check and card 4's constants and `tapestryPlanKeyWayForward`, and both come earlier.
  - Doc text: held. It extends `configreg/doc.go`.
- **Card 6**
  - Seam: held. `Result.Moved` is walked by no reflective test.
  - Imports: held. `hubreconcile`'s allowlist (`seam_enforcement_test.go:14-26`) already admits `configsync` and `configreg`.
  - New test tier: held. The new test goes in the tier1 `ensure_integration_test.go`.
  - Doc text: falsified (F1).
- **Card 7**
  - Seam: held. The new `Module.RunSettingsBlocks` field is not reflected over. `configreg_test.go:184` only reflects the `Spec` func name.
  - Order: held. `profiles-flat.yaml` is created in the card itself.
- **Card 8**
  - Seam: held. The `walkHooks` field is walked by no reflective test.
  - Imports: held. `boardengine` and `configreg` are both in `hubreconcile`'s allowlist.
  - New test tiers: held. tier0 beside `carry_test.go`, tier1 in the integration file.
  - Doc text: falsified (F1).
- **Card 9**
  - New test tier: held. `loomcli` has tier1 files such as `circling_integration_test.go`.
  - Imports: held. `hubgeom` already imports `hubreconcile`.
  - Doc text: falsified (F1, primary owner).
- **Card 10**
  - Re-sign: held. `CheckLocal`'s only callers are in `loomengine`, which the card names by package glyph.
  - Doc text: held. `configengine/doc.go` quotes no refusal text.
- **Cards 11–14:** no seam, tier or order invariant at stake. Held.

**F1 — MEDIUM, consistency — `internal/hubreconcile/doc.go` (cards 9, 8, 6)**
Card 9 makes a stale stamp trigger the walk from every verb that loads the loom or tapestry config, not only start verbs. The package doc still says otherwise, and no card rewrites it:
- `doc.go:3` says "Its one entry point is Ensure; a start verb calls it, through EnsureOrWarn, before loading any module config."
- `doc.go:33` says "A mid-merge skip leaves the stamp absent, so the next start verb retries."

Card 9 lists `hubreconcile/doc.go` only under **Uses**, and cards 6 and 8 only **Extend** it, which may add but never rewrite. Card 8 also adds a new skip cause (the run_settings carry), which line 33 does not name.

Evidence: `internal/hubreconcile/doc.go:3`, `:33`.
Fix: move `internal/hubreconcile/doc.go` to card 9's **Edit**, rewriting lines 3 and 33 to "a verb that loads a reconciled config".

**F2 — MEDIUM, consistency — `pattern/PATTERN-config-layers.md:7` (cards 2, 6)**
Line 7 reads: "no step removes an unknown base-layer key but the operator's `lym config reconcile` or `lym config &lt;module&gt; --unset &lt;key&gt;`."

Card 6 makes every walk remove old moved keys from hub base files: it renames them, and drops them when null or when the new path is set. Card 2 edits only three parts of this file: the opening line, the "Keys the template lacks" section and the "Config reinterpretation" rule. Card 14's Prosa pass is limited to key names, so line 7 stays false.

Evidence: `pattern/PATTERN-config-layers.md:7`, and `:77-78` ("Every other reconcile keeps such a key … That covers the binary-change walk").
Fix: card 2's Intent also amends line 7's rule to except keys a move table names.

**F3 — LOW, consistency — card 4, `internal/configreg#TestModules_EveryTemplateKeyClassified`**
Card 4's Intent says this test and `TestModules_LocalLayerRefusals` "run over the nested paths unchanged". The test's `wantLocalDenied` table literally lists the flat loom and tapestry keys:
- loom: `code_auto_accept`, `discussion_reviewer_fan`, …
- tapestry: `plan_auto_accept`, `plan_lenses`, …

Once `LocalDenied` derives from the nested Categories it must hold `code.auto_accept` and so on. A writer who takes "unchanged" literally fails card 4's own gate. The test is already an Edit target, so only the Intent text is wrong.

Evidence: `internal/configreg/configreg_test.go:175-176`.
Fix: the Intent says the `wantLocalDenied` loom and tapestry rows move to the nested paths.

Tool calls:
1. Grep of the reflective tests in `configreg`, `hubreconcile`, `configengine` and `configsync`, the `hubreconcile` seam test, and whether the test functions the cards name exist. The manifest held none of these.
2. Grep of the `doc.go` files in `loomcli`, `beamcli`, `webstercli`, `shuttleparity`, `hubgeom`, `hubreconcile`, `configreg`, `reviewbudget` and `configengine`, and of `pattern/PATTERN-config-layers.md`, for text the cards falsify. The manifest held no doc text.
3. Tests that read `PATTERN.md`, the package clause of the `configengine` tests, the `loomcli` tier1 files, and the classification test's tables. These were needed for the new-test-file and order checks.
4. `configreg_test.go:150-169`, `Module.LocalKeys` and the `pattern_test` functions. This confirmed which table carries loom and tapestry keys.
5. `internal/pattern` embed and the `patternentry` tests. This checked whether the `PATTERN.md` edits must be mirrored elsewhere; they need not be.
6. `internal/pattern/check.go` function list. This checked for format rules on a new PATTERN entry; none were binding beyond the gate.
