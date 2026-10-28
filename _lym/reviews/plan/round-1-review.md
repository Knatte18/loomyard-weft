---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/01-code-reviewer-fan-local.md (Tests bullets for TestLoadConfig_Refuses, TestModules_LocalLayerRefusals, TestPrepareRunSettings)
    summary: Card 1 plans three test additions whose behaviour existing or co-planned rows already pin - the local nosuchfan subtest, the admitted TestModules_LocalLayerRefusals row and the three refused retired-key rows
    fix: In 01-code-reviewer-fan-local.md, (1) delete the sentence of the TestLoadConfig_Refuses bullet that plans the sibling subtest writing a local code_reviewer_fan of nosuchfan, keeping the key-loop change; the refusal text is pinned by the existing row "unknown code_reviewer_fan" (internal/loomengine/config_test.go:610), and a local value reaching LoadConfig's merged config is pinned by the card's own new TestLoadConfig_Loads hub-stack row, while ValidateFanKeys runs on the merged config whatever layer the value comes from. (2) Remove the Edit line `internal/configreg#TestModules_LocalLayerRefusals` and the Tests bullet that adds its code_reviewer_fan admitted row; the test pins only keys no loader pins through its real load (internal/configreg/configreg_test.go:180), and code_reviewer_fan's local admission is pinned by the new TestLoadConfig_Loads row and its classification by TestModules_EveryTemplateKeyClassified's wantLocalDenied. (3) Delete the sentence of the TestPrepareRunSettings bullet that adds refused rows for plan_fan, discussion_fix_fan and plan_fix_fan; the existing row "a retired loom fan key is renamed by loom's Migrate and refused as a guard key" (internal/configreg/runsettings_test.go:97-101) pins that path, and the five keys staying guards is pinned by wantLocalDenied. Checked with lym loom validate-plan --overlay; it adds no blocking finding.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/01-code-reviewer-fan-local.md (Intent, the bound paragraph)
    summary: Card 1 says a wrong code_reviewer_fan name fails the pair's loom config load through ValidateFanKeys, but darn and pawl pairs read the key through configreg.SettingLookup with no fan resolution and meet the refusal only when the Code-Review round resolves its fan
    fix: In 01-code-reviewer-fan-local.md's Intent, after "a wrong name fails the pair's loom config load through `lenses.ValidateFanKeys`, whose refusal names the key and the fix", add a sentence saying this holds on a loom pair, while a darn or pawl pair reads `loom.code_reviewer_fan` through `configreg.SettingLookup` (internal/configreg/settinglookup.go), which resolves no fan, so there the wrong name is refused when the Code-Review round resolves the recorded fan, and the way forward is the same - set the pair's local entry to "" or to a fan lenses.yaml defines. No target change.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/04-repo-invariants-lens.md (Intent, Tests in internal/lenses)
    summary: Card 4 leaves internal/lenses/config_test.go's file header claiming the test pins each writer fan holding its reviewer fan's lenses, which repo-invariants as a reviewer-only lens makes false
    fix: In 04-repo-invariants-lens.md's Tests in internal/lenses, add a bullet saying config_test.go's file header (line 4, "each writer fan holding its reviewer fan's lenses") gains the exception, for example "each writer fan holding its reviewer fan's lenses but the reviewer-only repo-invariants". Add no target, since the card already edits members of config_test.go and a test file self glyph under Edit trips edit-test-file.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:way-forward
    location: _lym/plan/03-board-scope-lens.md and _lym/plan/05-code-review-lenses.md (Intent, recovery paragraph)
    summary: The cards' operator recovery, dropping the deleted lens name with lym config lenses --set, is rolled back while another fan still names a deleted lens, because setChecked undoes a write whose CheckLoad error text changes
    fix: In the recovery paragraph of 03-board-scope-lens.md and of 05-code-review-lenses.md, after "drop the deleted lens name from the fan with `lym config lenses --set` or the editor", add "fixing every fan that names it in one call, one --set per fan, since a --set that leaves another broken fan changes the load error and is rolled back". No target change.
---

### [LOW] Card 1 plans test rows whose behaviour is already pinned

**Location:** `_lym/plan/01-code-reviewer-fan-local.md`, the Tests bullets for `TestLoadConfig_Refuses`, `TestModules_LocalLayerRefusals` and `TestPrepareRunSettings`.

**Issue:** Three of card 1's test additions pin behaviour another row already pins.

- The local `nosuchfan` subtest asserts `loom config key "code_reviewer_fan" entry 1` and `set the entry to ""`.
  The existing row "unknown code_reviewer_fan" (`internal/loomengine/config_test.go:610`) already asserts both strings.
  `ValidateFanKeys` runs on `LoadConfig`'s merged config (`internal/loomengine/config.go:1225`), whichever layer the value came from.
  The card's own new `TestLoadConfig_Loads` hub-stack row already pins that a local value reaches that config.
- The admitted `code_reviewer_fan` row in `TestModules_LocalLayerRefusals` duplicates two other pins.
  That test's own doc limits it to keys "no loader card pins through its real load" (`internal/configreg/configreg_test.go:180`).
  The new `TestLoadConfig_Loads` row pins the same admission through the real load, and `wantLocalDenied` pins the classification.
- The refused rows for `plan_fan`, `discussion_fix_fan` and `plan_fix_fan` run the same path as the existing `discussion_fan` row (`internal/configreg/runsettings_test.go:97-101`): rename by Migrate, then the guard refusal.
  Each adds only another entry of the rename table.

The test-economy rule, and `PATTERN-test-speed`'s background (`pattern/PATTERN-test-speed.md:128-136`), treat a redundant test as a finding.
These are subtests and rows rather than new test functions, so the rating is LOW, not a PATTERN breach.

### [LOW] Card 1's stated bound holds only on loom pairs

**Location:** `_lym/plan/01-code-reviewer-fan-local.md`, the Intent's bound paragraph.

**Issue:** The card says a wrong name from the newly open routes "fails the pair's loom config load through `lenses.ValidateFanKeys`".
Only the loom and webster wiring calls `loomengine.LoadConfig` (`internal/loomcli/wiring.go:378`, `internal/webstercli/wiring.go:125`).
A darn or pawl pair reads `loom.code_reviewer_fan` through `configreg.SettingLookup`, which loads with `configengine.Load` and resolves no fan (`internal/configreg/settinglookup.go:51`).
So on D6's darn and pawl targets, a bad name is refused only when the Code-Review round resolves its fan.
The bound in D5 still holds: the value can name only a fan that `lenses.yaml` defines, or the round fails.
What the card gets wrong is where the refusal happens.

### [LOW] Card 4 leaves config_test.go's file header false

**Location:** `_lym/plan/04-repo-invariants-lens.md`, the Tests in `internal/lenses`.

**Issue:** `internal/lenses/config_test.go:4` says the file pins "each writer fan holding its reviewer fan's lenses".
Card 4 makes `repo-invariants` a reviewer-only lens that `writerFanGaps` admits, so the header stops describing the check.
The card names no change to it.

### [NIT] The `--set` recovery fails one fan at a time

**Location:** `_lym/plan/03-board-scope-lens.md` and `_lym/plan/05-code-review-lenses.md`, the recovery paragraph of each Intent.

**Issue:** `setChecked` undoes a write when the load fails with an error text other than the one before the write (`internal/configcli/configcli.go:269-296`).
`lenses.CheckLoad` joins every broken fan into one error.
A hub with two fans naming `goal-scope` that fixes one by `--set` gets that write rolled back.
The recovery the cards name works only in one call that fixes every such fan, or through the editor.

## Focus settled

- Lens counts and fan orders: the template defines 38 lenses today.
  Card 3 removes two and adds one, giving 37; card 4 adds one, giving 38; card 5 removes one, giving 37.
  Each card's `wantEntries` and `wantOrders` match the template it leaves.
  `way-forward` gains its emphasis line in card 4, the card where it leaves `review-plan`, and `repo-invariants` and `board-scope` stay exempt.
  So each card passes its own gate from the state the earlier cards leave.
- Fidelity: every Decision and Acceptance criterion is carried.
  - D1 by card 3; D2 by card 4; D3 and D4 by card 5; D5 by card 1; D6 by card 2.
  - D7: `docs/overview.md`, loom's `ConfigSpec` comment and `pattern/PATTERN-config-layers.md` by card 1; `runprofile/doc.go` by card 2; `docs/dotnet-hub.md` by card 5.
  - D5's retired-key admission: by the run-settings rows, with the local-file route resting on reconcile's existing migration.
  - Constraints: no card breaks one.
    No card names a stencil, and the local no-strip constraint is pinned by card 2's `TestReconcileAll_SparseLayers` row.
- Card 1's refusal rows: each reaches its refusal and no upstream check rejects it first (see the guard-reach report).
  Their redundancy is F1.
- Test economy:
  - `TestTemplateChecks_FlagBrokenTemplates` is not redundant: the template holds no lens that must fail, so only hand-built maps reach the negative D1 and D2 criteria.
  - The `TestReconcileAll_SparseLayers` row is not redundant: no existing row holds a former template value in a local file.
  - The `TestLoadConfig_Refuses` subtest is redundant, under F1.
- `plan:` handles: `emphasisLineGaps` and `TestTemplateChecks_FlagBrokenTemplates` are created in card 3.
  `writerFanGaps` is created in card 4, and card 4 edits or uses only handles card 3 created.
  No card uses a symbol a later card creates.
  The baseline `validate-plan --overlay {}` is clean.
- PATTERN fit:
  - `PATTERN-config-layers`: card 1 changes its background in the same card, as the Constraints ask.
  - `PATTERN-sparse-hub-base` and the lenses reconcile: template-only edits, no hub write.
  - No card breaks an entry.

## Rejected

- None of the fork findings were rejected.
