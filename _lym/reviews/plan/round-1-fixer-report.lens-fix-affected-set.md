```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: I re-ran each set-premise finding's search and checked that the fix covers every member.

**Findings left unchecked:** F1 got no repo search; I checked it only against the cards' new slot holders: the reach step's refresh, `lym gate mutate`, and `lym gate reach`, which takes no slot. F2 got no repo search either; I checked it only against each card's Create and Uses lists.

**Findings**

- **MEDIUM — card 4 (gate-config-reach), the Tests bullet on `TestLoadConfig`. The F3 fix covers only the loomcli helper; two existing `TestLoadConfig` rows whose expected `Reports` come from the template will fail.**
  - The changed text says only "the 'an unknown report' row's wanted refusal becomes the new message".
  - `internal/gateslot/config_test.go:303` is the row "a tuned webster without reports takes the template's". It expects `GateKey{Tiers: []int{0, 1}, Scope: "all", Reports: []string{"redundancy"}}`.
  - `config_test.go:318` is the row "a key's lints load in order". It expects `parent-align` with `Reports: []string{"redundancy"}`.
  - Both keys are tuned without `reports`, so config load fills in the template's value. Once card 4 makes that `[reach, redundancy]`, both expectations are wrong.
  - Card 4 should say these two rows' expected `Reports` become `[]string{"reach", "redundancy"}`.
  - `config_test.go:76-82` is `templateDefaults`, which card 4 already updates.
- F3, other members: the only test that string-replaces a template line holding `reports` is `gateConfigWithTwoTiers`, which card 4 now covers. `config_test.go:806-807` is migration input and does not match the template. The `batch:` literals in the loomshed and websterengine integration tests are self-contained configs.
- F4: the non-test files naming the redundancy bounds are `gateslot/doc.go`, `template.yaml`, `config.go`, `gatecli/redundancy.go`, `redundant.go`, `testmap.go`, `gatecli/doc.go`, `PATTERN-test-speed.md` and `docs/overview.md`. The cards' doc edits cover every prose one. `config.go` and the gatecli `.go` files hold code and their own comments, which the review's premise did not include.
- F11: every positional `[]site{...}` literal is in `internal/testcover/mutant_test.go`, an external test package. It cannot name the unexported `site`, so those literals use the test's own local type. `mutant.go:662` is the only production literal. No unkeyed literal of the production `site` breaks when it gains `callee`.
- F2: every production function a card creates is called from a main in that same card. Card 5 (reach step) is the first card where gatecli imports `testreach`, and it uses `Open`, `Reader.TestsReaching`, `Maps`, `SpawnMarked`, `StaleLinkers`, `Fresh` and `ReadPackageMap`. The guard in card 3 is a test file, so it adds no main import before that.

**Tool calls**
1. Grepped `redundancy_test_memory_mb` and `redundancy_budget_min` in non-test files: my context lacked the full set of docs and code naming the bounds (F4).
2. Grepped test files for `"redundancy"}` and grepped `site{` in testcover: my context lacked the tests that hard-code the template's reports (F3) and the `site` literals (F11).
3. Read `internal/gateslot/config_test.go:288-327`: I needed what rows 303 and 318 assert.
4. Grepped `Reports` in other packages' test files: to find assertions on gate-key reports outside gateslot. None were relevant.
