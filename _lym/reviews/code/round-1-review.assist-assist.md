---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:37
    summary: The contained spawn-boundary tests lack the required keep markers for their distinct assertions.
    fix: Add a //testtiming:keep directive immediately above TestAuditForkTranscript_SpawnBoundaryAgentCallExcluded naming its spawn-call exclusion and retained-activity assertions, and above TestAuditForkTranscript_MissingMetaFallsBackToCountingEverything naming its missing-metadata Agent count assertion. Preserve those assertions.
    origin: lens:code-tests
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:85
    summary: The new metadata classification test bypasses the public audit surface required by PATTERN-test-speed.
    fix: Move the new metadata classification cases into public AuditForks fixture coverage in internal/shuttleengine/claudeengine/audit_test.go, extending TestAuditForks_FullLayout to write sibling metadata and assert FreshSubagent for each case. Remove the new direct-helper classification test after preserving its cases there.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/audit.go:328
    summary: Readable metadata with no recorded subagent kind is classified as fresh and bypasses the self-check audit.
    fix: Require affirmative metadata evidence of a fresh subagent before setting FreshSubagent, retaining fork classification when neither discriminator records a kind. Preserve either positive fork discriminator as sufficient to classify a fork. Add empty-object, JSON-null and tool-ID-only cases to the public classification coverage requested in F2, all expecting FreshSubagent false.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/burlerengine/cluster.go:552
    summary: A named fresh delegate still fails the fix round through the unfiltered NamedSpawns tally.
    fix: Preserve the requested spawn kind alongside the named-spawn evidence in claudeengine and expose a provider-neutral tally for named fork requests. Use that tally for the fix-round named-fork refusal while preserving the existing all-named-spawns behavior for review and Webster callers. Add fix-audit cases accepting a named fresh delegate and rejecting a named self-check fork request, including one that lost inherited context because it was named.
    origin: lens:code-affected-set
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:85
    summary: No classification case independently pins the isFork discriminator when agentType is absent.
    fix: In the public classification table requested in F2, add metadata containing isFork true and toolUseId but no agentType, expecting FreshSubagent false. Keep the existing fork-type-without-flag case so each discriminator is tested independently.
    origin: lens:code-tests
---

### [BLOCKING] Preserve the distinct assertions with keep markers

**Location:** internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:37 and :68

**Issue:** The matching redundancy report lists both tests as contained by the new classification test.
Its only behavioral assertion is `FreshSubagent`, whereas these tests assert Agent counts and retained fork activity.
They should remain, but the rubric and `PATTERN-test-speed` require keep markers explaining those distinct assertions.
This is a review obligation; the advisory redundancy report does not enforce it mechanically.

### [BLOCKING] Exercise classification through the public audit

**Location:** internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:85

**Issue:** The new test calls `auditForkTranscript` directly at line 102.
`PATTERN-test-speed` requires helper behavior to be tested through the public surface that uses it.
`TestAuditForks_FullLayout` already creates the provider directory layout and invokes `AuditForks`.
The classification cases can use that existing file-contract fixture without a live provider or subprocess.

### [MEDIUM] Unknown metadata kinds escape enforcement

**Location:** internal/shuttleengine/claudeengine/audit.go:328

**Issue:** `{}`, `null` and metadata containing only `toolUseId` decode successfully into zero-valued discriminators.
The expression therefore sets `FreshSubagent` to true.
`auditFixRound` then excludes that transcript from write, nested-spawn and mutating-git checks.
This contradicts `ForkReport`'s stated conservative fallback when the provider records no kind.
Either `isFork:true` or `agentType:fork` alone correctly retains a fork; the defect is the absence of both kind signals.
The real incident metadata inspected records explicit fork and general-purpose kinds;
this finding concerns the accepted unknown-kind boundary, not a claim that those records were misclassified.

### [MEDIUM] Named delegates remain false fork violations

**Location:** internal/burlerengine/cluster.go:552

**Issue:** The shallow copy retains `NamedSpawns` while only the transcript slice is filtered.
`auditParentTranscript` counts every named Agent request, regardless of kind.
`forkViolation` rejects that aggregate even when every retained self-check fork is clean.
A named fresh delegate therefore still produces a hard failure under a policy now described as excluding fresh delegates.
The distinction must preserve rejection of a requested self-check fork that loses its inherited context through naming.

### [MEDIUM] Test the fork flag independently

**Location:** internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:85

**Issue:** Every fixture with `isFork:true` also supplies `agentType:fork`.
Removing the flag check from production would leave all classification assertions passing.
The focus directive explicitly asks about forks whose metadata omits either discriminator.
The fork-type-only case is covered, but the flag-only case is not.

## Scope and consumer assessment

The audit correction fits the board entry's writing-session option.
The existing self-check stencil already prohibits mutation and requires findings through the final message.
The summary considers and declines automatic revert and retry, which the entry asks to consider rather than require.
No scope finding follows from leaving those prompts unchanged.

The transcript enumeration already included every subagent before this diff.
Review-round auditing, Webster's batch and exit audits, write collection, and the CLI retain their existing transcript set.
The filtered fix-round count and unreturned-report warnings refer to self-check forks as their callers expect.
The existing write-violation row still checks that a self-check fork's write fails the round.

## Validation

Read the supplied diff pieces and commit messages for `22a2c25b767617b80815edf3f0748f816ac96796..HEAD`.
Inspected the incident's local metadata and extracted its delegates' write/edit calls with `jq`.
Those records support distinguishing legitimate fresh delegates from self-check forks.
No target files were changed, and no test suite was rerun within this report-only write surface.
