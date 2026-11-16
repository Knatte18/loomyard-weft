```yaml
lens: repo-invariants
outcome: reported
```

Task: repo-invariants lens. For each card I checked test seams and tier tags, doc or PATTERN text the card falsifies, and whether the card can pass its gate in card order.

**Per card, each invariant checked**
- Card 1:
  - Test-only-code guard: broken (F1).
  - `websterengine` `ScratchDir` doc comment: broken (F4).
  - Recipe text, `runplan/doc.go` and `websterengine/doc.go`: held.
- Card 2:
  - `configreg` registration and classification tests: held.
  - PATTERN-config-layers degrading list: held (Extends lists it).
  - `testmain_test.go` untagged support: held.
- Card 3:
  - Card order (`hubgeom` imports `weaverengine`, no cycle): held.
  - `lymdirs/doc.go` text: broken (F5).
- Cards 4 to 7: Webster readers become delegates, callers unchanged, no shared struct gains a field. All held.
- Card 8: stencil registry, `TestRegistry_MatchesOnDiskTree` and directive scans listed. Held.
- Card 9:
  - Gate keys `card` and `batch` described as Webster-only in PATTERN-test-speed and gateslot's doc: broken (F3).
  - The new verify-site calls in `batchGateSpec`: broken (F2).
- Card 10: held.
- Card 11:
  - `gatefixer` role doc: held (listed).
  - PATTERN-gate-slot "Verified tree" and "Gate fixer" text, and the `verifiedtree` tripwire's site list: broken (F2).
  - Gate key `webster` described in PATTERN-test-speed: broken (F3).
- Cards 12 to 14: no shared struct, no doc outside `weaverengine`. Held. The card 13 wiring of `Run` is the handler's.
- Card 15: `cmd/lym` registration, clitree, untagged `testmain`: held.
- Card 16: `shedadapters` doc and `docs/overview.md` listed. Held.
- Card 17: re-sign callers, including the `loomrecipe` shape table, listed. Held.
- Card 18: new `Env` fields with `//lym:seam` markers on the four loomcli wiring tests, batten and envkit; `nilLegal` and `intentionallyNil` listed. Held.
- Cards 19 and 20: `loomshed` and `loomcli` doc and overview listed. Held.
- Card 21: `decisionDeps` carries no seam marker (none exists today); `RunDeps.Deliver` is covered by card 18's marker. Held.
- Card 22: `PRReworkDeps` is unchanged; the keep reason is repointed. Held.
- Cards 23 to 26: held.

**Findings**

- **F1 — BLOCKING, consistency, card 1** (`runplan#ConflictRecordRel`): card 1 adds a production function that no `main` package reaches, and no card adds its allowlist entry.
  - Its only caller is `websterengine.ConflictRecordRel`, which is itself allowlisted as test-only (`cmd/lym/testonlycode_linux_test.go:61`).
  - The recipe names the path as YAML text, not a Go call, so nothing in production calls either function.
  - The test-only-code guard then fails, and card 1 cannot pass its batch gate.
  - Fix: add a `runplan.ConflictRecordRel` allowlist entry to `cmd/lym/testonlycode_linux_test.go` (a card target), or drop the function and have the test oracle join the `lymdirs` segments.
- **F2 — MEDIUM, consistency, cards 9 and 11**: Weaver adds plan-verify sites, and no card updates the background or the tripwire that list them.
  - The new sites are `runVerify` and the gate closures in `batchGateSpec`, all through `verifytree.Verify`.
  - `pattern/PATTERN-gate-slot.md` lists the verify sites only as Webster's, Webster-Burler-Fix's and the landing rows' ("The sites are the webster gate…"). Its `round`-fixer paragraph says Webster's fixers share one prompt body.
  - `cmd/lym/verifiedtree_test.go:18-22`'s `planVerifySites` names only Webster's files, so the tripwire never checks that Weaver's site passes a `Hold`.
- **F3 — MEDIUM, consistency, cards 9 and 11**: two docs describe the `card`, `batch` and `webster` gate keys as Webster's alone, and Weaver now reads them too.
  - `pattern/PATTERN-test-speed.md`'s "Gate keys" table gives their sites as "each Webster card", "each Webster batch" and "Webster's end gate".
  - `internal/gateslot/doc.go:219`'s "# Tier plan" section says which recipe row reads which key.
  - No card lists either file.
- **F4 — LOW, consistency, card 1** (`websterengine#ScratchDir`): its doc comment still says the scratch directory holds "the plan-conflict record and last-routed conflict" (`internal/websterengine/state.go`, the `ScratchDir` comment). Card 1 moves the record to `.lym/implementer/` but lists only `doc.go` and `runplan#Geometry`.
- **F5 — LOW, consistency, card 3** (`internal/lymdirs/doc.go`): the doc says `UsageDirName` and `RaddleDirName` are the only constants under the tracked `_lym` (lines 8-9). Card 3's `WeaverDirName` names `_lym/weaver/` too, and card 3 lists `lymdirs/doc.go` only under Uses.

Other lenses: none.

Tool calls:
1. Grep of `cmd/lym/verifiedtree_test.go`, `internal/gateslot/doc.go`, `PATTERN.md` and `internal/lymdirs/doc.go`: my context lacked the verify-site tripwire list, gateslot's tier-plan doc and the lymdirs doc text.
2. Grep of agentname and the role constants, the `//lym:seam` markers, the testonlycode allowlist, and the batten, loomcli, hubgeom and shedadapters docs: my context lacked the seam markers, the allowlist entry, and whether a closed role registry exists.
