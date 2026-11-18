---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/claudeengine/audit.go:328
    summary: The FreshSubagent derivation compares meta.AgentType to a repeated fork string literal although the package already declares forkSubagentType for that name, breaking PATTERN-code-conventions' Named constants rule.
    fix: In internal/shuttleengine/claudeengine/audit.go, replace the literal "fork" in the FreshSubagent expression with the existing constant forkSubagentType (declared in internal/shuttleengine/claudeengine/background.go:34), and widen that constant's doc comment so it also covers the agentType Claude Code records in a subagent's .meta.json. Keep the subagentMeta.AgentType field comment free of the literal or name the constant there.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/audit.go:328
    summary: A readable meta file that records neither isFork nor agentType (null, {}, or toolUseId alone) reads as a fresh subagent, so the fix-round audit skips that transcript's writes, contradicting ForkReport's rule that an unrecorded kind reads as a fork.
    fix: In auditForkTranscript, set FreshSubagent only on positive evidence of a fresh subagent - metaFound, meta.IsFork false, and meta.AgentType non-empty and not the fork type - so a meta with no recorded kind reads as a fork. Add rows for {} , null and a toolUseId-only meta, each wanting FreshSubagent false, to the classification table (placed per F4).
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/burlerengine/cluster.go:552
    summary: auditFixRound filters Forks but keeps the parent-wide NamedSpawns, so a fixer that names a fresh delegate still fails the round as a named-fork violation with a revert-the-fork way forward.
    fix: In internal/shuttleengine/claudeengine/audit.go auditParentTranscript, count named Agent calls whose subagent_type is the fork type separately, expose that count as a new provider-neutral ForkAudit field (for example NamedForkSpawns, documented in internal/shuttleengine/forkaudit.go), and in auditFixRound set selfCheck.NamedSpawns from that fork-only count so forkViolation judges only named fork spawns. Leave NamedSpawns and auditClusterRound unchanged for the review round and webster. Add a TestAuditClusterRound fix row where a named fresh delegate is clean and one where a named fork spawn is still an error, and cover the new count in the parent-transcript audit test.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:85
    summary: TestAuditForkTranscript_FreshSubagentReadFromMeta tests the unexported auditForkTranscript helper directly, while PATTERN-test-speed's Test economy rule asks for a helper to be tested through the public surface that uses it, and TestAuditForks_FullLayout already drives AuditForks over a fixture layout.
    fix: Move the FreshSubagent classification rows into public-surface coverage in internal/shuttleengine/claudeengine/audit_test.go - extend TestAuditForks_FullLayout, or add a table test beside it built on the same layout, so each row writes a subagent transcript with its sibling .meta.json and asserts the ForkReport.FreshSubagent AuditForks returns for it - and delete TestAuditForkTranscript_FreshSubagentReadFromMeta. Carry over every existing row plus the rows F2 and F5 add.
    origin: assist:assist
  - id: F5
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:89
    summary: No classification row pins the isFork term alone - every row with isFork true also has agentType fork - so deleting !meta.IsFork from the derivation fails no test.
    fix: Add a classification row (in the table F4 places) whose meta has isFork true, a toolUseId and no agentType, wanting FreshSubagent false, beside the existing ForkTypeWithoutForkFlag row.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:37
    summary: The redundancy report lists TestAuditForkTranscript_SpawnBoundaryAgentCallExcluded and TestAuditForkTranscript_MissingMetaFallsBackToCountingEverything as contained, yet each pins assertions its cover never checks, so each needs a testtiming keep marker.
    fix: Directly above func TestAuditForkTranscript_SpawnBoundaryAgentCallExcluded add the line //testtiming:keep pins the spawn-boundary Agent call's exclusion and the fork's own tool counts and report; directly above func TestAuditForkTranscript_MissingMetaFallsBackToCountingEverything (line 68) add //testtiming:keep pins that a missing meta file excludes no Agent call. If F4 moves the cover out of this file, re-check the report before adding them.
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: consistency
    location: internal/shuttleengine/engine.go:213
    summary: The ForkAuditor doc still says AuditForks reads the provider's record of fork subagents, while ForkAudit.Forks now holds every subagent with FreshSubagent telling them apart.
    fix: In internal/shuttleengine/engine.go, reword the AuditForks and AuditForksIncremental doc comments to say they report every subagent of the session, each ForkReport marking whether it is a fork or a fresh subagent.
    origin: lens:code-affected-set
  - id: F8
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/audit.go:28
    summary: The audit.go file header (lines 3-4) and the AuditForks and AuditForksIncremental docs still speak of fork subagents and every fork transcript, though the audit now reports and classifies every subagent transcript.
    fix: In internal/shuttleengine/claudeengine/audit.go, reword the file header's fork subagents and fork transcripts to subagents and subagent transcripts, and the AuditForks doc's every fork transcript and AuditForksIncremental's every fork transcript to every subagent transcript.
    origin: lens:code-affected-set
---

### [BLOCKING] Repeated fork literal beside the package's named constant

**Location:** internal/shuttleengine/claudeengine/audit.go:328

**Issue:** `FreshSubagent: metaFound && !meta.IsFork && meta.AgentType != "fork"` spells the fork subagent type as a literal.
The same package declares `const forkSubagentType = "fork"` at `internal/shuttleengine/claudeengine/background.go:34` for that name, used by the Agent-launch check at `background.go:281`.
`PATTERN-code-conventions` (Named constants): a name that can go stale is one named constant, declared once and referenced everywhere, never a repeated literal.
A listed-constraint violation is BLOCKING however small.

### [MEDIUM] A meta file with no recorded kind fails open

**Location:** internal/shuttleengine/claudeengine/audit.go:328

**Issue:** `readSubagentMeta` returns `found=true` for `{}`, `null` or `{"toolUseId":"…"}`, which leave `IsFork` false and `AgentType` empty, so the expression yields `FreshSubagent=true`.
`auditFixRound` then drops that transcript from the write, nested-spawn and git-mutation checks.
The diff's own contract at `internal/shuttleengine/forkaudit.go:39` says "A transcript whose kind the provider does not record reads as a fork", and the summary says the audit "stays strict when Claude Code records nothing".
Every real meta file on this machine records both keys for a fork (357 sampled) and `agentType` for every other kind, so the hole opens only on a Claude Code format change, but the change would be silent.
This settles focus item 1: every real shape classifies correctly today, including a fork missing one of the two keys; only a meta missing both reads wrong.

### [MEDIUM] Named fresh delegates still fail the round as named forks

**Location:** internal/burlerengine/cluster.go:552

**Issue:** `selfCheck := *audit` copies `NamedSpawns`, which `auditParentTranscript` counts over every named Agent call of the fixer, whatever its `subagent_type`.
`forkViolation` fails on `audit.NamedSpawns > 0` with "named forks silently lose inherited context" even when every retained fork is clean.
The diff's stated policy is that the fix-round audit neither checks nor counts a fresh subagent, yet a named delegate is still read as a self-check fork's defect, and the error's way forward, reverting what the fork wrote, does not fit.
This settles focus item 2 otherwise: the read-only contract stays enforced for self-check forks, a fork that spawns a delegate is still caught by its own `AgentCalls`, and the fork count and unreturned warnings now count self-check forks only, as `concludeFix` and `fix_forks` expect.

### [MEDIUM] The new classification test bypasses the public audit surface

**Location:** internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:85

**Issue:** The new test calls the unexported `auditForkTranscript` directly.
`PATTERN-test-speed`'s Test economy section says an unexported helper is tested through the surface that uses it, and a rule written there outranks the file's existing habit of calling the helper.
`TestAuditForks_FullLayout` (`internal/shuttleengine/claudeengine/audit_test.go:38`) already builds the project layout and calls `AuditForks`, and the classification is a file contract that layout can carry at tier0.

### [LOW] The isFork term is untested on its own

**Location:** internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:89

**Issue:** Rows `Fork` and `ForkTypeWithoutForkFlag` both carry `agentType: "fork"`, so removing `!meta.IsFork` from `audit.go:328` leaves every row passing.
Focus item 5 asks for a fork whose meta omits either discriminator; the omits-agentType case has no row.

### [LOW] Contained spawn-boundary tests need keep markers

**Location:** internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:37

**Issue:** The redundancy report lists both older tests as contained by `TestAuditForkTranscript_FreshSubagentReadFromMeta`, which asserts only `FreshSubagent`.
`SpawnBoundaryAgentCallExcluded` asserts `AgentCalls == 0`, the `Agent` and `Read` tool counts, `BashCommands` and `ReportReturned`.
`MissingMetaFallsBackToCountingEverything` asserts `AgentCalls == 1` without a meta file.
Each pins what its cover cannot see, so each is kept with a `//testtiming:keep` reason rather than folded.

### [LOW] ForkAuditor docs still describe fork subagents only

**Location:** internal/shuttleengine/engine.go:213

**Issue:** "AuditForks reads the provider's on-disk record of fork subagents" now disagrees with `forkaudit.go`'s widened `Forks` doc ("one report per subagent the parent spawned, forks and fresh subagents alike").

### [LOW] audit.go header and AuditForks docs still say fork transcripts

**Location:** internal/shuttleengine/claudeengine/audit.go:28

**Issue:** The file header (lines 3-4) speaks of "fork subagents" and "fork transcripts", and `AuditForks` and `AuditForksIncremental` of "every fork transcript".
The diff added one header line on the meta file but left these, so they now disagree with `auditForkTranscript`'s "reads one subagent's transcript" and the widened `ForkReport`.

## Holistic assessment

Scope (focus item 3): the board entry offers a prompt that states the no-mutation contract or a session allowed to write, and asks only that block, revert or retry be considered.
`contracts/stencils/burler/burler-fix-self-check.md:45` already states the forks' read-only rule and the final-message channel, and the halted run's writes came from two `general-purpose` delegates, not from a fork.
Exempting the fixer's delegates, which write as the fixer does, fixes the reported failure within the entry's licence, and the summary records why revert and retry were declined.

Other consumers (focus item 4): the review-round audit, `burlercli.resultEnvelope`, webster's batch, exit and write audits, and the fix-usage record read `Forks` exactly as before.
The enumeration already listed every subagent transcript before this diff; only the new field and the fix-round filter change, so the commit message's "unchanged" claim holds.

Tests (focus item 5): the fix-round row pins that a fresh subagent's write is neither a violation nor counted, the existing write row pins that a self-check fork's write still fails, and the meta table pins the missing and unparseable fallbacks; F4, F5 and F6 cover the gaps.

## Rejected

- lens:code-tests LOW, add a review-round row showing `auditClusterRound` still fails on a fresh subagent's write: the diff leaves `auditClusterRound` and `forkViolation` untouched, so this pins no behavior the diff adds or changes.
- assist:assist F1 at BLOCKING: merged into F6 at LOW; a keep marker is a rubric-directed test-economy mark, not a violation of a listed PATTERN entry's text.
- assist:assist F5 at MEDIUM: merged into F5 at LOW; it is a missing row for one term of a derivation whose other rows pass.
