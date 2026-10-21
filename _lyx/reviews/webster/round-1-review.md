---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/websterengine/summary.go:56
    origin: lens:code-affected-set
    summary: "AppendIntegrationTriage still writes a summary section saying the plan-level `## verify:` suite failed, though that section and the gate that ran it are retired"
    fix: "In internal/websterengine/summary.go change the triage section text to name the `final` gate, for example: The `final` gate failed, but webster's triage did not attribute the failure to this run. Update triageSectionHead in internal/websterengine/summary_test.go:39 to the same text. Leave the internal/summaryparser/summary_test.go fixtures alone, they are sample text."
  - id: F2
    severity: LOW
    class: consistency
    location: internal/configcli/configcli_integration_test.go:157
    origin: lens:code-affected-set
    summary: "Guard row named for weakening landing's publish_verify sets a retired key that now loads and is pruned, so it no longer exercises a live executable key"
    fix: "In the guardRows table at internal/configcli/configcli_integration_test.go:157 replace the row with one that sets a live executable landing key (the row below already uses deploy), or delete it as a duplicate of the deploy row, since both pin the same prime-only guard."
  - id: F3
    severity: LOW
    class: consistency
    location: internal/websterengine/runlevel_git_test.go:62
    origin: lens:code-affected-set
    summary: "Test fixtures still write the retired go_parallel key and rely on it being tolerated"
    fix: "Replace the go_parallel line with the tier_parallel entry (or drop it) in the fixtures at internal/websterengine/runlevel_git_test.go:62, internal/webstercli/verify_integration_test.go:43, internal/websterengine/verifygate_integration_test.go:33, internal/loomshed/verifygate_integration_test.go:145, internal/planglyph/glyphchain_test.go:1051 and internal/configcli/configcli_integration_test.go:362. Keep the go_parallel rows in internal/gateslot/config_test.go, which pin the retired-key behavior."
  - id: F4
    severity: LOW
    class: design
    location: internal/gateslot/tiers.go:118
    origin: lens:code-correctness
    summary: "decodeTier accepts a negative covers entry because it only refuses entries at or above the tier number, though the refusal text says tier numbers lower than N"
    fix: "In the covers loop of decodeTier in internal/gateslot/tiers.go refuse covered < 0 as well as covered >= number, with the same refuse(\"covers\", ...) message, and add a negative-entry row to the covers cases in internal/gateslot/config_test.go."
  - id: F5
    severity: LOW
    class: consistency
    location: internal/landingshed/verifygate.go:285
    origin: handler
    summary: "The tier-timeout halt names lyx config gate from the prime without the role clause every other gate refusal carries, against the plan's Shared Decisions"
    fix: "Append the same role clause gateslot's runlog.go:113 timeout line carries (a session that is not the prime reports status: FAILED and the orch runs it) to the timeout message at internal/landingshed/verifygate.go:285, and extend the test in internal/landingshed/verifygate_test.go that pins this message to assert it."
  - id: F6
    severity: NIT
    class: design
    location: internal/gateslot/tiers.go:67
    origin: lens:code-correctness
    summary: "decodeTiers discards the Atoi error for a tier key whose number overflows int, so the tier is stored under a number that tierKey does not map back to"
    fix: "In decodeTiers at internal/gateslot/tiers.go:67 check the strconv.Atoi error and return a refusal naming the key, as the key-pattern refusal does."
  - id: F7
    severity: NIT
    class: scope
    location: internal/gateslot/pool_test.go:307
    origin: lens:code-tests
    summary: "Subtest a count of 1 takes one slot as Acquire does repeats what the existing Acquire limit test and the two-slot subtest assert"
    fix: "Remove the subtest from TestAcquireSlots in internal/gateslot/pool_test.go, or fold its single-slot assertion into the two-slot subtest, since TestAcquire_TakesFreeSlotsUpToLimitThenRefusesAnother already pins one held slot and one holder."
---

## Declared PATTERN changes

Both declared entries land.
`pattern/PATTERN-gate-slot.md` carries the declared `PATTERN-gate-slot` text, and `pattern/PATTERN-test-tiers.md` carries the declared `PATTERN-test-tiers` text, each linked from a shortened `PATTERN.md` line as the Shared Decisions ask.
No commit in the range carries a `Pattern-Departure:` trailer.

### [MEDIUM] Triage section still names the retired plan-level verify suite

**Location:** internal/websterengine/summary.go:56
**Issue:** The summary section `AppendIntegrationTriage` appends says the plan-level `## verify:` suite failed.
Cards 13 and 16 retired that section and moved its job to the `final` gate, so a webster summary now names a check that no longer exists.

### [LOW] Guard row sets a retired landing key

**Location:** internal/configcli/configcli_integration_test.go:157
**Issue:** `publish_verify` loads and is pruned, so the row pins the prime-only guard through a key no gate reads.

### [LOW] Fixtures write the retired go_parallel key

**Location:** internal/websterengine/runlevel_git_test.go:62
**Issue:** Six fixtures still write `go_parallel`, which loads only because the loader tolerates it.

### [LOW] Negative covers entry accepted

**Location:** internal/gateslot/tiers.go:118
**Issue:** `covered >= number` is the only refused case, so `covers: [-1]` loads although the message says lower tier numbers.

### [LOW] Timeout halt lacks the role clause

**Location:** internal/landingshed/verifygate.go:285
**Issue:** The Shared Decisions give every refusal that names `lyx config gate` the role clause; this halt names it without one.

### [NIT] Overflowing tier key

**Location:** internal/gateslot/tiers.go:67
**Issue:** `tier99999999999999999999` passes `tierKeyPattern`, and the discarded `Atoi` error leaves a number `tierKey` does not map back to.

### [NIT] Redundant single-slot subtest

**Location:** internal/gateslot/pool_test.go:307
**Issue:** The one-slot case is covered by `TestAcquire_TakesFreeSlotsUpToLimitThenRefusesAnother`.

## Rejected

- A stale comment above `TestAcquireSlots` claiming process-global state while the test is parallel (lens:code-tests): no such comment exists at `pool_test.go:304`.
- `planparser/fabricref.go` `overviewSpans` still scanning the overview's `## verify:` body (lens:code-affected-set): the plan spec documents the scan, and "accepted and ignored" speaks to gating, so code and spec agree.
- Retired-input mentions in `internal/verifytree`, `internal/loomshed` (`plan-verify-retired`), `internal/gateslot` and the `verify_tiers` load tests: they are the intentional retired-key readers and their tests.
- `TierList` refusing a hub with no `tier0` even for an empty `publish` list (lens:code-correctness): the plan's use-time refusals intend it.
