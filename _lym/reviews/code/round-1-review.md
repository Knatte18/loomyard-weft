---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redfirst_integration_test.go#TestGateRedFirst
    summary: "The red-first verb's base and module refusals lost their only tests: the deleted mutate, reach and map integration tests were the only rows reaching the shared verbDiffBase (--base naming no commit, no parent branch) and the module-of-tiers refusal, and no red-first test asserts any of them now."
    fix: "Add a table-driven refusal test for `lym gate red-first --gate rf` beside TestGateRedFirst in internal/gatecli/redfirst_integration_test.go, using the seedRedFirstModule fixture. Add one row per refusal. Run from the prime with no --base, and assert exit 1, the text \"the worktree has no parent branch to diff from\" and the way forward \"pass --base\". Run with --base no-such-commit, and assert \"names no commit\" and \"pass the SHA of a commit\". Run with a gate.yaml whose tiers declare neither tier0 nor tier1 and --base HEAD, and assert \"the hub declares neither tier0 nor tier1\" and \"declare tier0 or tier1\"."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: "The two-way call and the gate blast radius leave out a cross-binary effect: after a new binary's `config gate --set` rewrites a gates entry without its `reports: []`, an older lym binary on the hub still fills the template's reports into that entry and runs the reach, redundancy and mutation steps again."
    fix: "In _lym/landing/summary.md under ## Merge danger, keep the door two-way only with a sentence naming this effect. Say that gate.yaml is read by every lym binary of the hub, that an older binary fills a missing reports list from its own template, and so that a gates entry rewritten by this binary turns the report steps back on for any older binary still running, such as a pinned run or another machine. Add that until every binary is updated, an entry is rewritten with an explicit `reports: []` or not rewritten at all."
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: "_lym/landing/summary.md#Hub config"
    summary: "The board entry removes the stored maps, but the maps already written stay on disk with no reader, writer or cleanup: the hub's shared maps under the board directory's .lym/testcover (56 MB here) and each task worktree's own .lym/testcover. The change description names no step to delete them."
    fix: "In _lym/landing/summary.md under ## Hub config, add a line naming the manual deletion of the board directory's `.lym/testcover` directory and of each worktree's own `.lym/testcover`, which no lym code reads any longer."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-test-speed.md:222
    summary: "The new bullet says `cmd/testtiming -redundancy` takes a tier's env and a gate slot as a tier run does, but testtiming reads no tier's env from gate.yaml. It holds one slot in every mode through gateslot.HolderEnv and sets LYM_TEST_TIER from its tags (cmd/testtiming/resources.go:130-141, main.go:10-13 and 30-32)."
    fix: "Replace the first line of the bullet at pattern/PATTERN-test-speed.md:222 with: \"`cmd/testtiming` holds one gate slot in every mode, with its Go flags from `gateslot.HolderEnv` and `LYM_TEST_TIER` from its tags.\" Then change the second line to open \"Its `-redundancy` mode runs every test alone under a coverage binary\", keeping the rest of that line."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/doc.go:43
    summary: "The diff removed every non-tier holder the gateslot doc named, so the doc lists none. PATTERN-gate-slot.md:15 still places a gate run in one of the non-tier holders that doc lists, and cmd/testtiming, which still acquires a slot with HolderEnv, is not named."
    fix: "In internal/gateslot/doc.go, before the line \"The non-tier holders hold one slot and set their own Go flags through HolderEnv.\", add a line naming the remaining holder: \"`cmd/testtiming` is one: it holds one slot for its whole run in every mode.\""
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatefixer/rowgate.go:205
    summary: "The spawnAndJudgeAgain doc comment still says inner runs under the entry's re-run mark. The mark was removed, and line 226 now calls waitThenRun with inner directly."
    fix: "In internal/gatefixer/rowgate.go:205, delete the clause \", inner run under the entry's re-run mark\", so the line ends \"returns inner's verdict once the fixer has ended.\""
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gateslot/config_test.go#TestLoadConfig
    summary: "The change description says a hub gate.yaml still setting a retired top-level key, such as redundancy_budget_min, still loads, but no gateslot row pins it. The removed rows pinned the old bounds, and only the reports-in-a-gates-entry case got a new row."
    fix: "Add a TestLoadConfig row in internal/gateslot/config_test.go whose hub file is baseKeys plus \"redundancy_budget_min: 10\\nmutation_max_mutants: 40\\n\", expecting the same config as baseKeys alone and no error."
---

### [MEDIUM] Red-first refusals lost their only tests

**Location:** internal/gatecli/redfirst_integration_test.go#TestGateRedFirst

**Issue:** `verbDiffBase` and `goModuleOf` moved into `redfirst.go` and `gomodule.go` unchanged. At base, the only rows reaching their refusals were in tests of the removed verbs: `mutate_integration_test.go:316` (`ABaseNamingNoCommit`) and the prime-with-no-base row beside it, `reachverb_integration_test.go:201`, and `testmap_integration_test.go:987` for "declares neither tier0 nor tier1". A grep of the current `internal/gatecli/*_test.go` finds none of these messages. The red-first verb still emits all of them. Deleting the removed verbs' tests therefore also deleted the only coverage of surviving behaviour, and no `Loosened-Check:` trailer names it. The focus asked about exactly this.

### [MEDIUM] Merge danger omits the cross-binary effect of dropping `reports`

**Location:** _lym/landing/summary.md#Merge danger

**Issue:** The rubric marks a two-way call as a finding when the diff changes a format another binary reads, or what a config setting means. `GateKey.Reports` leaves the loader, and the summary says a `reports` list goes when `config gate --set` rewrites the entry. For an older binary, though, a gates entry with no `reports` takes its own template's reports (base `TestLoadConfig` row "a tuned webster without reports takes the template's"). The hub's `gate.yaml` sets `reports: []` on most keys (`_bolt/_lym/config/gate.yaml`). So the first whole-entry rewrite by this binary turns the advisory report steps back on for any older binary on the hub, such as a run pinned to an earlier lym. The change description's Merge danger section does not name this.

### [LOW] Stored map data stays on disk

**Location:** _lym/landing/summary.md#Hub config

**Issue:** The board entry lists "The stored maps" among the things to remove. The diff removes every reader and writer and `lymdirs.TestCoverDirName`, but no cleanup touches the existing data: `_bolt/.lym/testcover` holds 56 MB, and `internal/cleanupcli` names no testcover path. The `.lym` directory is a cache, so this costs disk space and harms nothing. The change description should still say how to remove it.

### [LOW] PATTERN-test-speed misdescribes testtiming

**Location:** pattern/PATTERN-test-speed.md:222

**Issue:** The base sentence "The redundancy runs take a tier's `env`" described the gate's own map and report runs. The rewrite moved it onto `cmd/testtiming -redundancy`, which reads no tier `env`. `withGateSlot` passes `gateslot.HolderEnv(os.Environ(), lease, tier0Parallel)`, and every mode holds the slot, not only `-redundancy`.

### [LOW] gateslot doc's non-tier holder list is now empty

**Location:** internal/gateslot/doc.go:43

**Issue:** `pattern/PATTERN-gate-slot.md:15` still says a gate run may sit "in one of the non-tier holders `internal/gateslot`'s doc lists". After this diff the doc lists none, although `cmd/testtiming/resources.go:147-163` still acquires a slot as a non-tier holder.

### [LOW] Stale re-run mark clause in rowgate doc comment

**Location:** internal/gatefixer/rowgate.go:205

**Issue:** The focus named this line. The other doc comments in the file were updated, but `spawnAndJudgeAgain`'s still describes the removed `RerunMark` behaviour.

### [LOW] Loading a retired top-level gate key has no gateslot row

**Location:** internal/gateslot/config_test.go#TestLoadConfig

**Issue:** `decodeConfig` unmarshals leniently (`internal/gateslot/config.go:307`), and `configengine.warnUnknownKeys` warns on such a key. That is what the change description's design decision relies on, but only the generic configengine tests reach it. A gateslot row would keep a later strict decode from refusing hub files that still carry the eight removed keys.

## Rejected

- The allowlist entries for `proc.MemoryCapCoversTree` and its platform half in `cmd/lym/testonlycode_linux_test.go` are not a defect. The probe's last production caller was `redundantmutants.go:565`. The probe reads `proc`'s own state, so it cannot move to a testkit. The PATTERN-test-only-code background allows such an entry, and the `Loosened-Check:` trailer names the guard's report as its cause.
- `testreach.ChangedTests` comparing declaration text instead of a SHA-256 of it selects the same tests as at base. For test functions, `isTestFunc` excludes receivers and `init`, so the base's init ordinal and method naming never applied. Text equality equals hash equality bar a collision.
- A hub `gate.yaml` holding `reports` inside a gates entry loads on every layer and verb: `yaml.Node.Decode` into GateKey is non-strict, and `yamlengine.Prune` does not descend into the per-name `gates` map (`internal/yamlengine/layers.go:524`). So neither the load warning nor `config reconcile --layer default --apply` touches it. This matches the change description's Hub config section.
