```yaml
lens: repo-invariants
outcome: reported
```

Repo-invariants lens: each card checked for test seams, test-file tiers, doc text it falsifies, and card order.

**Per card, each invariant checked:**

- **Card 1 (gatefixer-core)**
  - Test seams: held. The package is new, and `testmain_test.go` is untagged and holds only `TestMain`. The new tests name tier 0.
  - Doc text: **broken**. The card registers fixer strands under the runproc roles `gate-fixer` and `verify-fixer`, which are not in runproc's named role set. Neither `internal/runproc/doc.go:4`, `internal/runproc/record.go:24-43` nor `pattern/PATTERN-run-process-owner.md` is listed (finding 1).
  - Card order: held.
- **Card 2 (gatefixer-prompt)**
  - Test seams: held. One stencil is registered and embedded with one `//go:embed` per file (`contracts/stencils/stencils.go:3,18`). A `roleOpeningStencils` row and a marker row are added.
  - Doc text: held. `gatefixer/doc.go` is extended.
  - Card order: held. The frame is created in the same card that maps `gate-fixer` to it.
- **Card 3 (webster-fixer-on-core)**
  - Test seams: held. No `//lym:seam` marker covers `FixVerifyDeps`.
  - Doc text: **broken**. The file headers of `render.go`, `strand.go` and `recoverbatch.go` describe the members this card deletes, and none is listed (finding 2). `websterengine/doc.go` is listed. `webstercli/doc.go:56` is not falsified.
  - Card order: held. Cards 1 and 2 create every `gatefixer` symbol the card uses.
- **Card 4 (gate-fixer-config)**
  - Test seams: held. The `configreg` template-coverage category is added for `fixer_timeout_min`.
  - Doc text: held. `gateslot/doc.go` is extended.
  - Card order: held.
- **Card 5 (module-fixer-keys)**
  - Test seams: held. The reflective template-tag test with `//lym:seam internal/loomengine#Config` and `#TapestryConfig` (`internal/loomengine/config_test.go:1081-1082`) is named under Uses, and the new tags land in the templates.
  - Doc text: held.
  - Card order: held.
- **Card 6 (row-gate-wrap)**
  - Test seams: held. Only new types are added.
  - Doc text: held. `gatefixer/doc.go` is extended.
  - Card order: held. It uses only card 1's members.
- **Card 7 (shedrecipe-fixer-element)**
  - Test seams: held. `Env.GateFixer` goes into envkit's `nilLegal`, which `walkNil` skips (`internal/testkit/envkit/envkit.go:49,631`), and into `intentionallyNil` for batten and loom.
  - Doc text: held for `shedrecipe/doc.go`. `loomshed/doc.go` gains no `LensReviewOnly` line, but `doc-go-unlisted` is a plan-gate check, so this lens does not flag it.
  - Card order: held.
- **Card 8 (loom-row-fixer-wiring)**
  - Test seams: held. `intentionallyNil` drops `GateFixer`, and `TestWire_EverySeamFilled` covers it.
  - Test-file tier: `TestPairSlugOf` is tier 1 while the card's other tests are tier 0, and no file is named. The one-tier-per-file rule holds as long as the implementer places the test in a tier-1 file; not flagged.
  - Card order: held.
- **Card 9 (goto-ends-fixers)**
  - Test seams: held.
  - Doc text: held as far as this lens judges. `loomcli/doc.go` is under Uses only; a missing line there is the gate's `doc-go-unlisted` check.
  - Card order: held. `pairSlugOf` comes from card 8 and `Roles()` from card 1.
- **Card 10 (writer-stencils-fixer-mode)**
  - Test seams: held. `twilldirective.Values` carries no seam marker.
  - Card order: held. `pairSlugOf` comes from card 8 and the `fixer` element keys from card 7.
- **Card 11 (card-gate-front)**
  - Test seams: held. `frontCall` and `detachRequest` are under no reflective test.
  - Card order: held. `IsFixerRole` comes from card 1.
- **Card 12 (card-gate-runner-fixer)**
  - Test seams: held. No reflective test covers `detachSeams`.
  - Card order: held. It uses card 4's `Fixer` and `FixerTimeout`, card 6's `NewStrandFixer` and card 2's `RenderPrompt`.
- **Card 13 (webster-fork-card-fixer-text)**
  - Test seams: held. No seam marker covers `BeginDeps` or `RecoverDeps`.
  - Card order: held. `pairSlug` comes from card 3 and `GateKey.Fixer` from card 4.

**Findings:**

1. **BLOCKING**, class `consistency`, card 1 (`gatefixer.Registrar`, `RoleGateFixer`, `RoleVerifyFixer`); card 3 inherits it through webster's `Register`.
   - Summary: the fixer strand is registered with runproc under the roles `gate-fixer` and `verify-fixer`, which are not in runproc's role set. No card adds them to runproc or edits the docs that enumerate the roles.
   - What it falsifies:
     - `pattern/PATTERN-run-process-owner.md` says a spawner registers "under a role the package names".
     - `internal/runproc/doc.go:4` lists the closed role set (`RoleDriver` … `RoleGate`).
     - `internal/runproc/record.go:24-43` calls those constants "The roles a registered process plays for its owner".
   - Evidence: the four sources above; card 1's Intent: "`runproc.Register(runProcDir, owner, runproc.ChildRecord(panePID, role, nil))`".
   - Fix: a card adds runproc role constants for the fixer roles and lists `internal/runproc/doc.go` and the role block of `record.go`, or registers under a runproc-named role.

2. **MEDIUM**, class `consistency`, card 3 (webster-fixer-on-core).
   - Summary: card 3 deletes `RenderVerifyFixColdPrompt`, `webster-prefix-verify-fix-cold`, `StrandLive` and `TurnEndedAfter` from websterengine. It lists `doc.go` but not the three file header comments that describe those members as webster's own, so after the card each header names deleted members.
   - `internal/websterengine/render.go:1,4,25,32,35` name `webster-prefix-verify-fix-cold.md` and `RenderVerifyFixColdPrompt`.
   - `internal/websterengine/strand.go:1` says the file "implements webster's own strand/spawn seam helpers: StrandLive and TurnEndedAfter".
   - `internal/websterengine/recoverbatch.go:1` describes RecoverAwait as running "over webster's own classification machinery — Classify/PollUntilTerminal/TurnEndedAfter/StrandLive".
   - Fix: add the three file self glyphs under card 3's `**Edit:**`.

Tool calls:
1. Grepped `//lym:seam` markers on the structs the cards extend, and runproc's role constants. The manifest had neither the seam markers nor runproc's `record.go`.
2. Grepped envkit's `nilLegal` and `FullEnv`, and the readers of runproc roles. I needed to know whether `Env.GateFixer` in `nilLegal` satisfies the seam tests, and the manifest lacked envkit.
3. Read `pairteardown/teardown.go` role checks and the `End`/`EndSparing` callers, to see whether any reader of a closed role set is affected by new roles. Teardown was outside the manifest.
4. Grepped `runproc/doc.go`, `loomcli/doc.go`, `loomshed/doc.go`, `shedrecipe/doc.go`, `webstercli/doc.go`, `gateslot/doc.go` and the recipes, for doc text the cards' changes falsify. These docs were not in the manifest.
5. Grepped the stencil embed and family registration and the PATTERN.md headroom marker, to check that card 2's new `gatefix` family and card 1's PATTERN entry pass the card gate. The manifest lacked both.
6. Grepped the PATTERN headroom lint, to check whether card 1's new PATTERN entry could trip the `pattern` lint on the card key. It did not settle the question; left unflagged.
