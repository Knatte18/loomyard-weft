---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: pattern/PATTERN-config-layers.md:7
    summary: "--unset removes a key the template lacks from a hub base or worktree file, which PATTERN-config-layers reserves to lym config reconcile alone, and the diff leaves the rule unchanged"
    fix: "In pattern/PATTERN-config-layers.md, change the Layers rule at line 7 (\"only `lym config reconcile` prunes unknown base-layer keys\") and the first sentence of \"Keys the template lacks\" at line 73 so that the operator's `lym config <module> --unset <key>`, through `configengine.Unset`, also removes such a key by name. Keep the ban on every automatic removal and the reason that follows it. If the one-line PATTERN.md entry is touched, keep it matching."
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/hubreconcile/doc.go:22
    summary: "The package doc still says `lym config reconcile --apply` is the one way to remove a key the template lacks, which --unset makes false"
    fix: "In internal/hubreconcile/doc.go:22, replace \"`lym config reconcile --apply` is the one way to remove such a key\" with a sentence naming both operator routes: `lym config reconcile --apply` and `lym config <module> --unset <key>`."
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/codexengine/prepare_test.go:333
    summary: "No test pins that LoadRefuseRaw's fallback still refuses an invalid refuse_raw when another gate key fails the full load; deleting its yaml.Unmarshal or decodeRefuseRaw check passes every test"
    fix: "Add a row to TestPrepare_InstallsGuardsFromItsInputs (codexengine/prepare_test.go) and to the refuse-list table of TestPrepare_WritesArtifactsAndReturnsConsistentLaunch (claudeengine/settings_test.go). The gate file holds the scope-less `gates: {harden-live-copies: {tiers: [1]}}` entry and an invalid list, `refuse_raw: [\"go  test\"]`, and each row wants the refusal before any artifact. Add a second row to each table with `refuse_raw: 3` beside the same entry, wanting the same refusal."
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: decision
    location: internal/configreg/configreg.go:150
    summary: "The widened RetireHardenLiveCopies keeps the reinterpretation ID the merge base already shipped, so no hub that ran that binary and wrote its marker ever gets the scope-less removal"
    fix: "Register the gate module's reinterpretation under a new ID in internal/configreg/configreg.go:150. Declare it as a new constant beside RetiredCopiesReinterpretationID in internal/gateslot/reinterpret.go, for example \"gate-harden-live-copies-scopeless-retired\", so a hub past the first marker runs the rewrite once more; the rewrite returns a file with no entry unchanged. Update the summary's design decision to match. If the parent prefers the manual `--unset` route for marked hubs, defer to it and record that call with lym loom decision add."
    origin: lens:code-scope
  - id: F5
    severity: LOW
    class: design
    location: internal/gateslot/config.go:285
    summary: "When refuse_raw is invalid and an earlier key also fails, LoadRefuseRaw refuses with the full load's first error, which names the gate key the launch never reads and not refuse_raw"
    fix: "In LoadRefuseRaw, when decodeRefuseRaw(file.RefuseRaw) fails, return that error, wrapped with the gate config way forward the full load gives, such as through withConfigWayForward(err, ConfigFiles(dirs), \"\"). Keep returning loadErr for the parse and merge failures."
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: consistency
    location: internal/configcli/configcli.go:310
    summary: "An --unset that newly breaks the module load is refused with \"would not load with this value\" and \"way forward: --set the key to a value it takes\", which misdirects a removal"
    fix: "Make newLoadFailure's text fit both writes, for example \"the %s config would not load after this write: %w; way forward: keep the key, or --set the keys it leaves invalid to values the loader takes\". Alternatively, pass the wording in from the layerWrite so --set keeps its current text."
    origin: lens:code-correctness
  - id: F7
    severity: LOW
    class: design
    location: internal/gateslot/reinterpret.go:50
    summary: "The rewrite treats an empty decoded scope as the sign that an entry stops the gate load, so an entry with tiers that do not decode and a present scope is removed as naming no scope, while one with an invalid scope is left"
    fix: "In RetireHardenLiveCopies, decide the removal on whether the entry decodes and its scope is one decodeGates takes (card for card or batch only, diff or all), not on copies.Scope == \"\". Remove and flag every entry that fails that check, with a rewrote line that says the entry stopped the gate config loading rather than that it names no scope. Add rows to TestRetireHardenLiveCopies for `{tiers: [oops], scope: diff}` and `{tiers: [1], scope: coverage}`."
    origin: assist:assist
  - id: F8
    severity: LOW
    class: design
    location: internal/configcli/configcli_test.go:573
    summary: "The --unset rows never show that the key left a worktree file, that a list with one missing key removes nothing, or that an absent file is refused"
    fix: "In TestDispatchSet's row \"unset on a worktree file syncs once\", read the worktree loom.yaml after the dispatch and assert discussion_timeout_min is gone. In TestDispatchHubWideBoard, add a row with unsetFlags {\"labels.old\", \"labels.absent\"} that wants code 1 and the file still equal to seeded. Add a row with absent true and unsetFlags {\"labels.old\"} that wants code 1 and the \"does not exist\" error."
    origin: lens:code-tests
  - id: F9
    severity: LOW
    class: design
    location: internal/configcli/configcli.go:166
    summary: "The new refusals \"module required with --unset\" and \"--resolved and --unset are mutually exclusive\" have no test row"
    fix: "Add a TestDispatchSet row with no args and unsetFlags {\"discussion_timeout_min\"}, wanting code 1 and \"module required with --unset\". Add a TestPrintResolved row with --resolved beside --unset, wanting \"--resolved and --unset are mutually exclusive\", next to its existing --set, --print and --layer rows."
    origin: lens:code-tests
  - id: F10
    severity: LOW
    class: design
    location: internal/gateslot/reinterpret_test.go:25
    summary: "TestRetireHardenLiveCopies checks only the --set substring of the flagged way forward, so the line-48 mutation survivor (BilledTier in setLive) and the new --unset step pass unchecked"
    fix: "In TestRetireHardenLiveCopies, also assert the literal \"one tier below tier3\" (the billed tier written out) in every flagged line. Assert \"`lym config gate --unset gates.harden-live-copies`\" in the flagged lines of the SeveralLiveTiersAreFlaggedAndLeft and BilledTierIsFlaggedAndLeft rows."
    origin: lens:code-tests
  - id: F11
    severity: LOW
    class: consistency
    location: internal/configcli/configcli_test.go:1238
    summary: "TestDispatchSetProfiles is listed as contained by TestDispatchLayers but pins profile bytes and profile validation errors TestDispatchLayers never asserts, and carries no keep mark"
    fix: "Add `//testtiming:keep pins the profiles module's written bytes and its profile validation refusals, which TestDispatchLayers does not assert` on the line directly above `func TestDispatchSetProfiles`."
    origin: assist:assist
  - id: F12
    severity: LOW
    class: consistency
    location: internal/configcli/configcli_test.go:308
    summary: "The diff adds \"--unset\" to TestConfigLong's wanted substrings, a new assertion on the static prose of the help text"
    fix: "Remove \"--unset\" from the wants slice in TestConfigLong and revert its doc comment line to name only the --set flag; F9's refusal rows cover the flag's behaviour."
    origin: assist:assist
  - id: F13
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:50
    summary: "The Merge danger blast radius names shuttle alone, but the diff also changes every config module's --set write path and adds a removal from hub files, which commits in the Bolt checkout"
    fix: "In _lym/landing/summary.md's ## Merge danger, widen the blast radius to name shuttle launches, config writes through lym config (configcli, configengine, yamlengine) on every layer with their hub commits and fabric sync, and the gate reinterpretation that deletes a hub gate.yaml entry. Keep the one-way door."
    origin: assist:assist
  - id: F14
    severity: NIT
    class: design
    location: internal/yamlengine/layers.go:30
    summary: "A key named twice, as in --unset a --unset a, is removed on the first pass and then reported as \"no key a in the file\", refusing the whole write"
    fix: "In RemoveKeys, skip a key already seen in keys before calling removeKey, for example by tracking seen keys in a map."
    origin: lens:code-correctness
---

### [BLOCKING] --unset breaks the reconcile-only removal rule

**Location:** pattern/PATTERN-config-layers.md:7

**Issue:** PATTERN-config-layers states "only `lym config reconcile` prunes unknown base-layer keys" (line 7), and line 73 says "Only the operator's `lym config reconcile` removes a key the running template lacks, from a hub base or worktree file."
`configengine.Unset` removes any named key whatever the template holds, a retired key included, and the diff's own doc text says so in `configengine/doc.go`, `configcli/doc.go` and the reconcile Long help.
No commit carries a `Pattern-Departure:` trailer, and the round declares no PATTERN change.
The feature is what the board asks for, so the rule text must change with it.

### [MEDIUM] hubreconcile doc still names reconcile as the one removal

**Location:** internal/hubreconcile/doc.go:22

**Issue:** The diff updated the matching sentences in configcli and configengine but left "`lym config reconcile --apply` is the one way to remove such a key" here.

### [MEDIUM] The launch fallback's refuse_raw checks are untested

**Location:** internal/shuttleengine/codexengine/prepare_test.go:333

**Issue:** Both engine tables pin the scope-less key launching with a valid list and a file that does not parse.
Neither reaches the fallback with an invalid `refuse_raw`.
The summary and both `Prepare` doc comments state that guarantee as what keeps a broken file from launching without the guard.

### [MEDIUM] Hubs already marked never get the scope-less removal

**Location:** internal/configreg/configreg.go:150

**Issue:** At the merge base the gate module already registers `RetiredCopiesReinterpretationID`.
The hubreconcile walk runs each reinterpretation once per hub behind its board marker.
A hub that ran the merge-base binary had its scope-less `{tiers: [1]}` entry flagged and kept, and its marker is now written, so the fixed rewrite never runs there.
That is the case the board entry reports.
The summary names the choice and sends such hubs to `--unset`, but the board's goal point asks the reinterpretation to cover the entry.
Whether to re-key is the author's or the parent's call.

### [LOW] An invalid refuse_raw refusal names the wrong key

**Location:** internal/gateslot/config.go:285

**Issue:** `decodeConfig` checks `gates` before `refuse_raw`, so with both broken, `loadErr` names the gates key.
The launch then refuses over a key its doc says it never reads, and the real cause goes unnamed.

### [LOW] newLoadFailure words an unset as a set

**Location:** internal/configcli/configcli.go:310

**Issue:** `setChecked` and `setHubWide` now apply any `layerWrite`, so a removal that exposes an invalid lower value, or leaves a dependent key invalid, gets "with this value" and a `--set` way forward.

### [LOW] Scope as a proxy for a failing entry

**Location:** internal/gateslot/reinterpret.go:50

**Issue:** `decodeGateKey` returns the zero `GateKey` on any decode error, losing a present scope.
`{tiers: [oops], scope: diff}` is then removed with the rewrote line "which names no scope", which is false.
`{tiers: [1], scope: coverage}` stops the gate load as surely, yet it is left in place and flagged.
The doc comment justifies the removal by the load stopping, not by the scope being absent.

### [LOW] Unset rows miss the file, atomicity and absent-file cases

**Location:** internal/configcli/configcli_test.go:573

**Issue:** The worktree row asserts the sync count and envelope and never reads the file.
Every refusal row passes one key of a present file, so neither `RemoveKeys`' "nothing is removed" rule nor `Unset`'s absent-file refusal is reached.

### [LOW] Two new refusals have no row

**Location:** internal/configcli/configcli.go:166

**Issue:** "module required with --unset" (`dispatch`) and "--resolved and --unset are mutually exclusive" (`resolvedRefusal`) are new messages that no test reaches.

### [LOW] Flagged way forward under-asserted

**Location:** internal/gateslot/reinterpret_test.go:25

**Issue:** The mutation report's survivor at reinterpret.go:48 changes `BilledTier` in `setLive`.
The test's `wayForward` constant stops before "below tier%d", so the survivor lives, and no row checks the new `--unset` step of the scoped flag.

### [LOW] Contained test kept without its mark

**Location:** internal/configcli/configcli_test.go:1238

**Issue:** The redundancy report lists `TestDispatchSetProfiles` as contained by `TestDispatchLayers`.
`TestDispatchLayers` never dispatches the profiles module, so the profile bytes and the `unknown profile`, `JSON object` and `run profile` refusals are the candidate's alone.
The rubric then asks for the keep mark.

### [LOW] Static help prose asserted

**Location:** internal/configcli/configcli_test.go:308

**Issue:** The rubric's test-economy rule counts a new assertion on static text as a finding, and "--unset" in `wants` asserts only that the Long string holds the word.

### [LOW] Blast radius names less than the diff touches

**Location:** _lym/landing/summary.md:50

**Issue:** The `layerWrite` seam reroutes every `--set` write of every module.
`--unset` adds a removal that commits hub files, and the reinterpretation now deletes a hub gate.yaml entry.
"shuttle" covers none of these.

### [NIT] Repeated unset key refuses the write

**Location:** internal/yamlengine/layers.go:30

**Issue:** `removeKey` mutates the root in place, so the second pass of a repeated key finds nothing and the whole write is refused naming a key the file held.

## Review notes

- The amended board entry matches the record's `## Goal`, and each of its three points has a change that builds it.
- The `Root-Cause:` trailer on 4ca9fdfb7 names the engines refusing `Prepare` on any `LoadConfig` failure, which the diff removes.
  The one on 1feb3c612 names the rewrite judging by tiers alone, which the diff changes.
  Both hold.
- The launch still refuses a file that does not parse or merge, and the stale-key fallback keeps the guard.
  The warning goes through `logger.Warn`, which reaches stderr at the default threshold and the durable trace sink.
- `--unset` takes the same `setModule` route as `--set`: the hub write guard on the default and machine layers, the commit through `setHubWide`, the fabric sync on the worktree layer and the `setChecked` load check with rollback.
- The docs the focus names, `docs/overview.md`, `docs/shared-libs/configengine.md` and the changed doc.go files, match the code, apart from F2's sentence outside them.

## Rejected

- code-scope F2, that every launch now tolerates the bad key and not only sessions that run no gate: the board's point is that a no-gate session must launch, and a gate-running session still refuses at its gate call with the full loader's way forward. Applying the one read to every launch is the minimal form of that point.
- The assistant's ask to assert the fallback's warning in a launch test: the warning is a log record, which the rubric's mutation guidance counts as not reasonably killable, and F3 pins the behaviour that matters.
- The assistant's ask for local and machine `--unset` rows and a rollback row: those paths are the shared `setLocal`, `setMachine`, `setChecked` and `setHubWide` code that the existing `--set` rows already pin, and the new `layerWrite` only swaps the write call.
- The assistant's BLOCKING severities for the test gaps F3, F8, F9, F10 and F11: they break no PATTERN constraint as written, so they keep the severities above.
- Indentation drift from `yaml.Marshal` in `RemoveKeys`: every other yamlengine writer, `Prune` among them, marshals the same way, and `TestDispatchSetProfiles` shows the same output from `--set`.
