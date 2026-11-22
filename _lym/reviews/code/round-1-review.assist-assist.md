---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: pattern/PATTERN-config-layers.md:7
    summary: Named unknown-key removal contradicts the unchanged reconcile-only removal rule.
    fix: "Update the removal rule at lines 7 and 73 to permit explicit lym config --unset through configengine, retaining the ban on automatic unknown-key pruning; update internal/hubreconcile/doc.go:22 to name both operator routes."
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/shuttleengine/codexengine/prepare_test.go:333
    summary: Invalid refuse_raw and fallback warning behavior have no launch regression assertions.
    fix: "Extend the existing launch tables with an invalid prefix and invalid refuse_raw shape, checking refusal before artifacts; capture and assert the warning level and underlying gate error in an existing stale-key launch case."
    origin: lens:code-tests
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/configcli/configcli_test.go:573
    summary: Removal tests omit repeated-key atomicity, resolved inheritance, rollback and resolved-flag exclusion.
    fix: "Extend existing dispatch/layer rows to assert removed bytes and inherited values, repeated removals and an absent later key preserving bytes, and rollback with no sync when removing an override exposes an invalid lower value. Add local and machine unset rows, the machine guard case, and a resolved/unset refusal row to TestPrintResolved."
    origin: lens:code-tests
  - id: F4
    severity: BLOCKING
    class: design
    location: internal/gateslot/reinterpret_test.go:104
    summary: Retirement assertions miss the mutated tier boundary and the new removal command.
    fix: "Assert the literal below tier3 in flagged results and the --unset gates.harden-live-copies command in retained scoped-entry rows of TestRetireHardenLiveCopies."
    origin: lens:code-tests
  - id: F5
    severity: BLOCKING
    class: consistency
    location: internal/configcli/configcli_test.go:1238
    summary: The retained contained profile test lacks its required keep annotation.
    fix: "Add //testtiming:keep immediately above TestDispatchSetProfiles, naming its profile addition/replacement bytes and profile-specific validation errors that TestDispatchLayers does not assert."
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/gateslot/reinterpret.go:79
    summary: A tier decode error discards a present scope and makes retirement delete a scoped entry.
    fix: "Read the retired entry's scope independently of tier decoding, retaining and flagging undecodable scoped entries; add a row with tiers [oops] and scope diff to TestRetireHardenLiveCopies."
    origin: lens:code-correctness
  - id: F7
    severity: MEDIUM
    class: consistency
    location: internal/configcli/configcli_test.go:294
    summary: The new help substring assertion pins static prose instead of command behavior.
    fix: "Remove the newly added --unset expectation from TestConfigLong; verify the flag through the command refusal row requested in F3."
    origin: lens:code-tests
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:50
    summary: The declared shuttle blast radius omits config writes across every module and layer.
    fix: "Expand Merge danger's blast radius to cover configcli/configengine/yamlengine writes across module layers, hub commits and worktree sync, as well as gate retirement and shuttle launches. Keep the one-way door classification."
    origin: handler
---

### [BLOCKING] Reconcile-only rule is stale

**Location:** pattern/PATTERN-config-layers.md:7

**Issue:** The rule and its Keys the template lacks section reserve unknown-key removal to reconcile.
The new `configengine.Unset` deliberately adds another route, as the board requests.
The diff updates config documentation but leaves that governing rule and `internal/hubreconcile/doc.go:22` unchanged.
This is a rule/documentation omission forced by the authorized feature, not unrequested feature scope.

### [BLOCKING] Launch fallback lacks refusal and warning checks

**Location:** internal/shuttleengine/codexengine/prepare_test.go:333

**Issue:** Both engine tables cover the valid stale-key fallback and malformed YAML.
Neither exercises the new fallback's invalid `refuse_raw` decoding or prefix rejection.
The old invalid-prefix rows exercise `LoadConfig`, which does not validate the fallback branch.
Neither launch table asserts the warning required by the goal.
These are observable behaviors without coverage under the repository's test-economy rule.

### [BLOCKING] Removal contracts lack observable assertions

**Location:** internal/configcli/configcli_test.go:573

**Issue:** The worktree unset row asserts sync and the envelope but never reads the resulting file.
The hub rows remove one nested entry or reject one absent entry.
They do not exercise repeated keys, partial-removal failure atomicity, or inherited values and rollback.
The layer test supplies no unset flags, leaving local and machine removal unexercised.
The command's new resolved/unset exclusion also lacks a refusal row.
Existing set checks do not exercise removal exposing a lower-layer value.

### [BLOCKING] Generated retirement warning is under-asserted

**Location:** internal/gateslot/reinterpret_test.go:104

**Issue:** Assertions check the tier-list prefix and set command only.
They accept a wrong numeric boundary in the generated warning at `reinterpret.go:48`.
That is the reported surviving mutation, and the test can pin the independent expected boundary.
They also accept omission of the new unset recovery command for retained entries.
This is generated operational output, not a static documentation assertion.

### [BLOCKING] Keep the distinct profile test explicitly

**Location:** internal/configcli/configcli_test.go:1238

**Issue:** The supplied redundancy report lists the test as contained by `TestDispatchLayers`.
Its exact profile bytes and profile validation errors are distinct assertions, so deletion would lose coverage.
`PATTERN-test-speed` requires a keep annotation for this retained candidate.

### [MEDIUM] Decode failure becomes deletion authority

**Location:** internal/gateslot/reinterpret.go:79

**Issue:** An entry `{tiers: [oops], scope: diff}` fails decoding into the integer tiers field.
The helper returns a zero `GateKey`, losing the present scope.
The caller then takes the scope-less removal branch and incorrectly reports that the entry named no scope.
`hubreconcile.applyReinterpretations` persists the removal and its marker.
The changed contract says differing scoped entries remain and are flagged.

### [MEDIUM] Static help check is not flag coverage

**Location:** internal/configcli/configcli_test.go:294

**Issue:** Adding `--unset` to the help substring list asserts literal prose in `buildConfigLong`.
It does not check Cobra registration or execution.
The review rubric explicitly excludes new static-prose assertions from acceptable test coverage.

### [MEDIUM] Merge danger understates affected configuration

**Location:** _lym/landing/summary.md:50

**Issue:** The summary declares only `shuttle` as its blast radius.
The diff also changes shared writes for every config module and adds removal across selected layers.
Those routes include hub commits and worktree fabric sync, beyond session launching.
The one-way classification itself is consistent with persistent configuration removal.

## Review evidence

The board body's three goals match the decision record; each has implementing changes.
Both Root-Cause trailers describe the defects their changes address.
The warning is delivered through the logger's stderr and durable sinks.
Ordinary stale-key fallback retains the guard, and unmergeable input still refuses.
The shared removal path retains layer selection, hub guards, commit/sync routing and load checks.
No waits or sleeps were added to the changed tests.

The attempted tier0 gate reported a read-only filesystem failure registering its hub runproc lock.
No completed test result was obtained; this review does not claim a test pass.
No target files were changed.

## Rejected

- The test lens's BLOCKING severity for the help-prose assertion is reduced to MEDIUM.
  The rubric forbids it, but the cited PATTERN's static-prose ban specifically names stencils.
- Deleting the contained profile test is rejected because its assertions differ from its covering test.
- The unchanged reinterpretation ID is not a new defect: the summary explicitly directs marked hubs to manual unset.
