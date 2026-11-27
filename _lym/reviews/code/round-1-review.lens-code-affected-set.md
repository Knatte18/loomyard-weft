```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I listed what reads or describes it and checked whether the diff left it right.

**Changes and their affected sets**

- **The `//lym:only` marker and `FindOnlyTests` / `ModuleOnlyTests`**
  - Readers and docs: `impactset/doc.go`, `gatecli/only.go`, `gatecli/doc.go` (Held tests), the `cli.go` Long help, `template.yaml`, and the background file `pattern/PATTERN-test-speed.md`. All covered, except the doc line under LOW 2.
  - The `PATTERN.md` entry for `PATTERN-test-speed` adds `//lym:only`. Covered.
- **`Targets.InsideGuards` and `skippedInside`**
  - The `impactset/doc.go` line, the `gatecli/doc.go` guard-step paragraph (its old "Bound:" is gone), the `PATTERN-test-speed.md` guard bullet and the `targets_test` rows are all updated. Covered.
- **`gateslot.ScopeMarked` and `UpkeepGateKey`**
  - The `decodeGates` refusal text and `config_test`'s "an unknown scope" row, `templateDefaults`, `template.yaml`, the `GateKey.Scope` doc, `gateslot/doc.go`, and the key table in `PATTERN-test-speed.md`. Covered.
  - `goReadGates` (`gateslot/tiers.go:153`) leaves out `upkeep`. Its doc says these are the keys Go verify sites read, and no Go verify site reads `upkeep`, so this is no finding.
- **The `held` field of `greenEntry` and the signatures of `covered` / `newestWholeAncestor`**
  - Callers are `subtract`, `carriesOverInert` and `diffBaseGroups`; tests are `green_test`. All updated.
  - The `gatecli/doc.go` green paragraph and the `scope.go` `diffBaseGroups` doc are updated. Covered.
- **The board-tidy upkeep step (`Deps.UpkeepCall`, `TidyPromptInput.UpkeepCall`, the two fragments, the `{{.upkeep}}` marker)**
  - The template's header comment, the stencil registry, `tidy_test`, the `beamcli` wiring and the `dobbyengine/doc.go` bullet. Covered, except the doc wording under LOW 1.
  - The template's "What you never do" list says never to "File or promote a task". The step files notes, and a note is not a task, so the rule is not contradicted.
  - `bug` is a board label (`beamcli/dobbydeps_integration_test.go:472`), so "a `bug` note" together with "Pick each note's labels with `lym board labels`" holds.
  - The verbs `lym board find`, `labels` and `upsert` exist (`boardcli/cli.go:325`, `:417`, `:562`). Covered.
- **Retiring `create-caller-later`**
  - The check, its spec entry in `loom-plan-checks-spec.md`, the sentence in `loom-plan-spec.md`, the plan-cards block, the plan-review rubric, the `pattern/PATTERN-test-only-code.md` background, and the `cmd/lym` test that kept the two lists equal are all removed or updated.
  - No reference to `create-caller-later`, `TestSupportDirectories` or "first production caller" is left in the repository.
  - The helpers it used (`hasTestPrefix`, `handleUnit`, `diskPathForRef`) still have other callers. Covered.
- **The `//lym:only` markers on the `cmd/lym` tests**
  - Every `cmd/lym` guard without `//lym:watch` now carries `//lym:only parent-align finalize`, except the reachability guard, which carries `upkeep`.
  - The watched guards stay unmarked, `TestCSharpFixtures_CompileWithDotnet` among them. This matches `PATTERN-test-speed.md` and the commit messages.
  - The doc comment of `crosscompile_test.go` is updated. Covered.
- **Declared PATTERN changes**: this round's directive declares none, so nothing is raised.
- **Commit messages**: no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers. The subjects agree with the diff.

**Findings**

1. **LOW · consistency** · `internal/dobbyengine/doc.go:204-207`
   - The new upkeep bullet says the agent runs the upkeep checks "first". The stencil makes them step 3, after the tidy steps, and `Deps.UpkeepCall`'s own doc says "before its report".
   - The bullet was also inserted above the line "Past `board_tidy_timeout_min` dobby ends the agent and posts a no-report line". That line now reads as part of the upkeep bullet, not the spawn bullet it belongs to.
   - Evidence: `internal/dobbyengine/doc.go:204-207`, `contracts/stencils/beam/beam-template-board-tidy.md` (step 3 `{{.upkeep}}`, then step 4), `internal/dobbyengine/loop.go` (`UpkeepCall` doc).
2. **LOW · consistency** · `internal/gatecli/doc.go:266`
   - The new Held-tests line names the config key as bare `` `gates` ``. `PATTERN-config-key-names` asks for the full `gate.gates` in a comment or doc.
   - The pre-existing line 33 of the same doc uses the same short form, so this follows local precedent but breaks the PATTERN rule.
   - `pattern/PATTERN-test-speed.md` writes it in full in the same diff ("A marker naming a key `gate.gates` does not hold").
   - Evidence: `internal/gatecli/doc.go:33`, `:266`; the `PATTERN-config-key-names` entry in `PATTERN.md`.

**Tool calls**

1. Grep for `goReadGates`, the `testSupportPackages` comment, stale create-caller text, lists of gate keys, board-tidy stencil references, board-tidy role settings and the board verbs: the reading list had none of these readers outside the diff.
2. Grep of `PATTERN-gate-slot.md`'s key list, the dobby template defaults, board kinds and labels, and the `TiersValue` tests: needed to check the gate-key lists, the tidy timeout and the label/kind wording.
3. Read the whole board-tidy template, `PATTERN-test-only-code.md`, and the short `gates.` key names in the changed files: needed to test the stencil's "never do" rules and the key-name rule.
4. Grep of how `gatecli/doc.go` already names `gates`, the dobby config module, and `bug` as a label: needed to judge whether the short key name breaks local convention.
5. Grep of the shellguard rows for `lym gate test`, plus `dobbyengine/doc.go:198-210`: needed to check the board-tidy role can run the gate call (no row found) and where the new doc bullet sits.
