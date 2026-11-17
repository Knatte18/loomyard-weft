```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens over cards 1–9. For each card I checked seam tests and tier tags, doc text a card makes untrue, and whether the card can pass its own gate from the state earlier cards leave.

**Per card**

- **Card 1**
  - Seam/tier: held. No reflective `//lym:seam` test in heddleengine, shuttleengine or the engine packages covers `Passage`, `Standing` or `Target`. The new tests state tier0.
  - Doc: held. It lists heddleengine `doc.go` and shuttleengine `doc.go`. The engine `doc.go` lines about passages (codexengine `doc.go:65`, claudeengine `doc.go:521-526`) stay true after the re-sign.
  - Order: held. Every caller of `Renderer.Passages` and `passagesOf` is in `Show`, `Passages` or `TestShowAndPassagesReadTheShippedLayerFirst`/`TestRender`; render_test.go:226, 754 and 793 are all inside those two tests. The reader install at `cmd/lym/main.go:180` is in `newRoot`, which the card lists. The Codex refusal's item name (`developerinstructions.go:49`) and its test row (`prepare_test.go:415`) are both edited by the card.
- **Card 2**
  - Seam: held. No reflective seam test covers `UserSetup`, `InstructionItem` or `OmittedHook`, so the new fields need none.
  - Doc: held. It lists shuttleengine `doc.go`, whose sweep text (doc.go:71-81) is the text it changes.
  - Order: held. `fakeEngine.PrepareHook` exists (`startup_test.go:222`).
- **Card 3**
  - Seam: held. claudeengine importing heddleengine for `Marker` passes shuttleengine's seam scan, which bars only concrete-engine imports (`seam_enforcement_test.go:2`). codexengine already imports heddleengine (`instructionfiles.go`).
  - Doc: held (claudeengine `doc.go` listed).
  - Order: held. `ListedSkills`, `Plugin` and `Hook` come from card 2.
- **Card 4**
  - Doc: held (codexengine `doc.go` listed).
  - Order: held. `ErrHookUnrun` comes from card 2, and `Plugin` is filled by card 3.
- **Card 5**
  - Seam: held. `NewWithSeams` adds no import beyond the allowlist in `internal/shuttleengines/seam_enforcement_test.go`.
  - Doc: held. shuttleengines `doc.go` and the provider-seam background are listed.
  - Order: held. All constructor call sites are listed, at shuttleengines.go:200, 228, 294, 307 and 319, plus `build`/`keepWarmWith`.
- **Card 6**
  - Seam: held. The new package imports no concrete engine.
  - Tier: held. Each test states tier0.
  - Order: held. It uses only card 2 and card 5 symbols.
- **Card 7**
  - Tier: held, tier0.
  - Order: held. `newParityFixture`, `PrepareEach`, `Compare`, `Unmet` and `seatTables` come from card 6.
- **Card 8**
  - Doc: held. `docs/overview.md:639` holds the shuttle verb list, and the card extends it.
  - Order: held. `internal/shuttlecli/resolve_verbs_test.go` holds a fixed row table with no every-verb enumeration, so a new verb forces no row there.
- **Card 9**: held. It adds a PATTERN entry and its background only.

**Findings**

None. The lens found no broken invariant that no card lists.

**Tool calls**

1. `//lym:seam` markers and reflective seam tests in the shuttle and heddle packages: the manifest did not cover seam tests.
2. Seam allowlists, and passage, sweep and run-dir wording in `pattern/` and the `doc.go` files: needed for the doc-falsification check.
3. Engine import allowlists and the "heddle passage" item naming: needed to check the card 3 import and the card 1 rename.
4. shuttleengine's seam scan rules and existing heddleengine importers under shuttleengine: needed to confirm claudeengine may import heddleengine.
5. Callers of `Passages` and `passagesOf`, the function enclosing `main.go:180`, and the verb lists in tests and `docs/overview.md`: needed for card 1 and card 8 order.
6. Which test functions hold `render_test.go:754` and `:793`, and whether `resolve_verbs_test` enumerates every verb: needed to confirm the card 1 test list and the card 8 verb coverage.
