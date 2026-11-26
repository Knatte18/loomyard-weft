```yaml
lens: repo-invariants
outcome: reported
```

Task: repo-invariants lens over cards 1–7. For each card I checked test seams, doc text the card makes false, and card order.

**Per-card invariant list**

- **Card 1 (run-watch-setting)**
  - Test seams: held. No reflective test covers `Config`, and `defaultsWith` is edited. `TestCheckLocalValues` goes into an existing test file, so no new tier tag is needed.
  - Docs: held. The `beamsetting` package doc, `Config`'s and `CheckLocalValues`' comments, the `tasksetting.go` header, the per-task bullet in `dobbyengine/doc.go:136-139` and `pattern/PATTERN-config-layers.md:125` are all listed and edited.
  - `LoadConfig`'s doc (`internal/dobbyengine/config.go:99-102`) says "A negative value is refused". It stays true, but it would not mention the new `gate_hold_min` minimum of 1. This is a NIT, recorded below.
  - Card order: held. The card depends on no other card, and no golden lists beam.yaml keys: the only file naming them is `template.yaml`, which the card edits.
- **Card 2 (run-view-history)**
  - Test seams: held. No reflective test covers `RunView` or `ChildReading`; the `reflect.DeepEqual` checks in `internal/beamcli/dobbypairanchor_test.go:238,335` compare projections, not whole structs.
  - Docs: held. `DobbyRuns`' doc lists some of what it carries but never claims the list is complete.
  - Card order: held.
- **Card 3 (run-watch-engine)**
  - Test seams: held. The new state map is tolerant, and `TestRunWatch` presumably goes into a new tier0 harness file, which the card does not name.
  - Docs: **broken**. The done-run-clear bullet at `internal/dobbyengine/doc.go:190-191` becomes false, and the card lists `doc.go` only under `Extends`.
  - Card order: held. It uses the `RunWatch*` constants from card 1 and the fields from card 2.
- **Card 4 (run-watch-wiring)**
  - Test seams: held. No reflective test covers the `Deps` wiring.
  - Docs: held. `internal/beamcli/doc.go:224-234` (the seams section) names no notice, so nothing there becomes false. `orch-role.md` is edited.
  - Card order: held. It comes after card 3.
- **Card 5 (gate-holder-marker)**
  - Test seams: held.
  - Docs: held. `gateslot/doc.go` is edited. `gatecli/doc.go:260` says the holder names the tier, which stays true. `pattern/PATTERN-gate-slot.md:19` covers the non-tier holders, which the card leaves unchanged.
  - Card order: held.
- **Card 6 (gate-hold-engine)**
  - Test seams: held. `TestGateHolds` presumably goes into a new tier0 file, unnamed by the card.
  - Docs: held. The package keeps its own `GateHolder` type, so the import allowlist is respected: `internal/proc` is already allowed in `seam_enforcement_test.go`, and `gateslot` is not needed. The told-geometry note in `doc.go` stays true.
  - Card order: held. It uses card 3's `RunWatchState` and card 1's `Config` fields.
- **Card 7 (gate-hold-wiring)**
  - Test seams: held. `TestDobbyGateHolders` is tagged tier 1, and the repository has no rule that tier-1 test files carry `_integration` in their name (see `dobbypairanchor_test.go:199`).
  - Docs: held. `beamcli/doc.go` says nothing about which packages it imports.
  - Card order: held. It comes after cards 5 and 6.

**Findings**

1. MEDIUM, consistency.
   - Location: `_lym/plan/03-run-watch-engine.md`, `**Extends:** internal/dobbyengine/doc.go`.
   - Summary: card 3 makes `clearDoneRuns` also wait while a run's `Watches` entry owes a line. That makes the done-run-clear bullet false: it says "A run is cleared only when … neither its landing notice nor its board-tidy count is still owed."
   - Why the card cannot fix it as written: `Extends` only allows adding text, so fixing the bullet means rewriting existing content, and the card does not say to.
   - Fix: move `internal/dobbyengine/doc.go` from `**Extends:**` to `**Edit:**` on card 3, and have the Intent say the done-run-clear bullet also names an owed watch line.
   - Evidence: `internal/dobbyengine/doc.go:185-191`, `internal/dobbyengine/donerun.go` (the doc of `clearDoneRuns`).
2. NIT, consistency.
   - Location: `_lym/plan/01-run-watch-setting.md`, `internal/dobbyengine#LoadConfig`.
   - Summary: the Intent has `LoadConfig`'s doc comment name `run_watch`, but not the new `gate_hold_factor` (must not be negative) and `gate_hold_min` (at least 1) refusals. The doc would still say only "A negative value is refused".
   - Fix: the Intent says `LoadConfig`'s doc names both range refusals.
   - Evidence: `internal/dobbyengine/config.go:99-102`.

**Tool calls**

1. Searched the tests of beamcli, dobbyengine, battencli and gateslot for reflective seam tests over the structs the cards extend; the manifest held no such test.
2. Searched `dobbyengine/doc.go`, `gateslot/doc.go`, `gatecli/doc.go`, `PATTERN.md` and `pattern/` for text on the behaviour the cards change; the manifest held no doc.go text.
3. Read the `reflect.` uses in `dobby_test.go` and `dobbypairanchor_test.go`, to tell whether they are seam tests.
4. Searched for tests asserting the state record's keys, read `dobbyengine/doc.go:180-225` (the done-run clear section), and searched the gate-slot pattern for its holder rule; the manifest lacked these spans.
5. Ran `git show --stat` on the settings-golden commit and searched for golden files listing beam.yaml keys, to judge whether card 1 can pass its own gate.
