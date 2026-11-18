---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

## Changes

- F1: `audit.go` compares `meta.AgentType` to `forkSubagentType`; the constant's doc in `background.go` now also covers the agentType in `.meta.json`; the `subagentMeta.AgentType` comment names the constant.
- F2: `FreshSubagent` is set only on positive evidence: meta found, `IsFork` false, `AgentType` non-empty and not the fork type.
  The `auditForkTranscript` doc says so.
- F3: `auditParentTranscript` counts named fork spawns into the new `ForkAudit.NamedForkSpawns` (documented in `forkaudit.go`).
  `auditFixRound` sets `selfCheck.NamedSpawns` from it.
  `NamedSpawns`, `auditClusterRound` and webster are unchanged.
  Rows added: a named fresh delegate is clean and a named fork spawn is an error in `TestAuditClusterRound`;
  the count is covered in `TestAuditForks_FullLayout` and the new `TestAuditForks_NamedForkSpawnsCountOnlyNamedForks`.
- F4: the classification table moved to `TestAuditForks_FreshSubagentReadFromMeta` in `audit_test.go`, driven through `AuditForks`;
  `TestAuditForkTranscript_FreshSubagentReadFromMeta` is deleted.
  Every old row is carried over, plus rows for `{}`, `null`, a toolUseId-only meta and the F5 rows.
- F5: rows `ForkFlagWithoutAgentType` and `ForkFlagWithOtherAgentType` pin the `isFork` term.
  The second was added after a self-check lens showed the first passes without `!meta.IsFork`.
- F6: `//testtiming:keep` lines added above `TestAuditForkTranscript_SpawnBoundaryAgentCallExcluded` and `TestAuditForkTranscript_MissingMetaFallsBackToCountingEverything`.
  The cover moved out of that file, but each test still pins assertions no other test makes, so the markers stay accurate.
  The redundancy report was not re-run.
- F7: `ForkAuditor`, `AuditForks` and `AuditForksIncremental` docs in `engine.go` describe every subagent, each `ForkReport` marking fork or fresh.
- F8: `audit.go` file header, `AuditForks` and `AuditForksIncremental` docs say subagent transcripts.

Gate repair: the gate refused comment lines the round added that wrapped at a fixed column or reached 120 characters.
They were rewritten one sentence per line in `cluster.go`, `audit.go`, `engine.go`, `audit_test.go` and `audit_parentfacts_test.go`, with no change of meaning.

Commits: `dbcd8d99` (F7), `7779410a` (F1-F6, F8, in one commit because they share files), and the self-check commit after them.
The gate (`lym gate test` on `shuttleengine`, `claudeengine`, `burlerengine`) passed after both fix rounds.

## Self-check

- fix-code-correctness, LOW, F5 row passes without `!meta.IsFork`: fixed (row `ForkFlagWithOtherAgentType`).
- fix-code-correctness, LOW, named call without `subagent_type` unpinned: fixed (test pins it, field doc states it).
- fix-consistency, LOW, stale "fork transcript" in the `AuditForksIncremental` doc, the read-error message and `ForkReport` comments: fixed.
- fix-consistency, LOW, `engine.go` cumulative-facts list omits `NamedForkSpawns`: fixed.
- fix-proposal, MEDIUM, a named call with no `subagent_type` passes the fix-round audit: fixed-differently.
  The proposed change, counting an omitted type as a fork, was not made:
  every fork launch in this repo names `subagent_type` explicitly (`background.go:281`, `parent-spawns.jsonl`),
  and counting an omitted type would fail a fixer's untyped named delegate, the defect F3 removes.
  The reading is documented on the `NamedForkSpawns` field and pinned by a test call with a name and no type.
- fix-proposal, LOW, field doc silent on an untyped named call: fixed (same change).
- fix-affected-set, LOW, `audit.go` "fork transcript" wording: fixed (duplicate of the consistency finding).
- fix-affected-set, LOW, `wait.go` finalize doc says "fork subagents": fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
