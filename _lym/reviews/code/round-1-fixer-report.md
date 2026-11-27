---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposed test, a chain that reaches any FreshSubagent report, still leaves unchecked a fresh subagent that a self-check fork spawned, whose commit would then get the same false no-target-change way forward. The exemption is decided by the subagent at the head of the spawner chain instead."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The proposed line leaves out other readers of the same audit: burler's cluster-round audit, and webster's run-exit audit and run-write history. The line names them too."
  - id: F6
    disposition: fixed
---

## Changes

- **F1** (fixed differently): `ForkReport.SpawnedBySubagent` is replaced by `SpawnerTranscriptPath`, which `AuditForksIncremental` fills.
  `auditFixRound` follows the spawner chain to the subagent the session spawned, and exempts the report only when that one is a fresh subagent.
  Not done: the proposed "chain reaches a `FreshSubagent` report" test, since `internal/burlerengine/cluster.go` would then still skip a fresh subagent under a self-check fork.
  `cluster_test.go` has a row for a write by a fork under a self-check fork and one for a commit by a fresh subagent under it; the exempt row now covers a two-level chain.
  The `auditFixRound` comment, `burlerengine/doc.go` and the landing summary say the same.
- **F2**: `audit_nested_test.go` gains a session-spawned fork, `agent-direct`, that replays a session Bash call and Agent call.
- **F3**: the missing-spawner-transcript error uses `%v`, with a comment saying why; the test asserts it does not match `fs.ErrNotExist`.
- **F4**: the self-check stencil line is replaced as proposed.
- **F5** (fixed differently): the blast radius line in `_lym/landing/summary.md` names burler's fork audits, webster's record-batch and run-exit audits and run-write history, and shuttle's fork-authorized run finish. Not committed, being a run record.
- **F6**: the `ForkAudit` doc is corrected.

Commits: `49e83b0e1` (F1, F2, F3, F6), `afd53ce72` (F4), and a follow-up on `burlerengine/doc.go` from the self-check.

`lym gate test` passed on `internal/burlerengine`, `internal/shuttleengine/...`, `contracts/...` and `internal/websterengine`, and again on `internal/burlerengine` after the self-check edit.

A later gate finding named wrapped comment lines in `audit_nested_test.go` and `forkaudit.go`.
Each is now one sentence on one line, committed as a comment-only follow-up.
`lym gate test` passed on `internal/shuttleengine/...` and `internal/burlerengine` after it.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `burlerengine/doc.go` "only the self-check forks" clashing with the added sentence: fixed.
- fix-affected-set, LOW, blast radius line omitting the cluster-round and run-exit audits: fixed.
- fix-proposal, LOW, the same blast radius omission (merged with the one above): fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
