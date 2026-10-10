# Card 6 — contract-evidence-names-the-writer

**Create:**
- `plan:internal/websterengine#unclearedContract` -> `type unclearedContract struct`
- `plan:internal/websterengine#noteWroteLast` -> `func noteWroteLast(writer string) string`

**Edit:**
- `internal/websterengine#contractFileStatus` -> `func contractFileStatus(geom Geometry, writes RunWrites, path string) (contract, cleared bool, writer string, err error)`
- `internal/websterengine#contractSplit`
- `internal/websterengine#splitContractPaths`
- `internal/websterengine#contractDeleteClause` -> `func contractDeleteClause(uncleared []unclearedContract, rerun string) string`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#AcceptPendingAudit`
- `internal/websterengine#checkPendingFindings`
- `internal/websterengine#pendingPathsWayForward`
- `internal/websterengine#ArchiveRunAfterReset`
- `internal/webstercli#websterCLI.acceptAuditCmd`
- `internal/webstercli/doc.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/contractevidence_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/archivereset_test.go`
- `internal/websterengine/findingtext_test.go`
- `internal/webstercli/acceptaudit_test.go`

**Delete:**
- `internal/websterengine#noteForkWroteLast`

**Uses:**
- `internal/websterengine#RunWrites`
- `internal/websterengine#latestSucceededWrite`

**Intent:** Count a recovery session's writes in the contract-file evidence and name the last writer's kind (decision D3).
`contractFileStatus` clears a present contract file only when Master's latest successful write is later than every fork write and every recovery write to it; its new `writer` return is `"a fork"` or `"a recovery session"`, whichever made the latest such write at or after Master's, and empty when cleared.
`unclearedContract` pairs a path with that writer; `contractSplit.Uncleared` becomes `[]unclearedContract`, filled by `splitContractPaths`.
`contractDeleteClause` and `noteWroteLast` say "a fork wrote" or "a recovery session wrote" per path, so a recovery-kept finding never reads "a fork wrote"; `noteWroteLast(writer)` replaces the deleted `noteForkWroteLast` constant at its two call sites.
`RecoverSpawnOrAttach`, `AcceptPendingAudit`, `checkPendingFindings` and `pendingPathsWayForward` adopt the new shapes; the `rm <path>` step and the rerun they name are unchanged.
`RecoverSpawnOrAttach`'s refusal prefix "failed on contract file(s) a fork wrote last" becomes "failed on contract file(s) a fork or a recovery session wrote last", the clause after it naming each path's writer.
Reword the doc comments that say a contract file "a fork wrote last" (`checkPendingFindings`, `pendingPathsWayForward`, `ArchiveRunAfterReset`, `contractSplit.Uncleared`, `contractDeleteClause`) to "a fork or a recovery session wrote last".
Reword the `Long` text of `acceptAuditCmd`, the archive guard sentence in `internal/webstercli/doc.go`, and the contract-file sentences of `internal/websterengine/doc.go` to say "a fork or a recovery session wrote last".
In `refusal-spec.md`, the rows whose Trigger or Way forward say "a contract file a fork wrote after Master's last write" or "a fork wrote last" (the recover-batch, accept-audit, `--fresh`, `reset --to start` and run-exit audit rows) say "a fork or a recovery session of this run".

Tests: add rows to `TestContractFileStatus` for a recovery write after Master's (not cleared, writer "a recovery session"), a recovery write before Master's (cleared) and a fork and a recovery both after Master's (the later one named).
Add a `recovery wrote last` case to `TestRecoverSpawnOrAttach_ContractFileEvidence` asserting the refusal names a recovery session and does not contain "a fork wrote", and keep the fork case asserting "a fork wrote".
Grep the repository for "a fork wrote" and "after Master's last write" and update every test asserting the old wording, including `archivereset_test.go`, `findingtext_test.go` and `acceptaudit_test.go`.

**ImpactSummary:** Two unexported re-signs and one field type change with all five readers in websterengine, one constant replaced by a function; refusal wording changes in two packages and the spec.
