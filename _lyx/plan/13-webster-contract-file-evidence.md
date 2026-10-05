# Card 13 — webster-contract-file-evidence

**Create:**
- `plan:internal/websterengine#contractFileStatus` -> `func contractFileStatus(geom Geometry, writes RunWrites, path string) (contract, cleared bool, err error)`
- `internal/websterengine/contractevidence_test.go`

**Edit:**
- `internal/websterengine#AcceptPendingAudit`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#pendingPathsWayForward`
- `internal/websterengine#pendingAuditFindingsError`
- `internal/websterengine#runExitAuditCrossCheck`
- `internal/websterengine#freshPendingDrop`
- `internal/websterengine#Run`
- `internal/webstercli#websterCLI.acceptAuditCmd`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/auditledger_test.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/acceptaudit_test.go`

**Uses:**
- `plan:internal/websterengine#RunWrites`
- `plan:internal/websterengine#loadRunWrites`
- `plan:internal/websterengine#wayForwardSteps`
- `internal/websterengine#OutcomePath`
- `internal/websterengine#checkSuspectPaths`

**Intent:**
D5: a fork-contract-write finding becomes checkable, instead of every weft path landing in `unverifiable` and leaving `--fresh`, which discards every batch, as the only way forward.

`contractFileStatus`, in a new `internal/websterengine/contractevidence.go`, is the one evidence function.
`contract` is true only when `path` resolves to this run's `outcome.yaml` or `summary.md` (`OutcomePath`, `summaryparser.Path` over `geom.WebsterDir`, compared canonically).
A contract path is `cleared` when the file is absent, or when the latest `writes.Master` event for that path with `Succeeded` true is later than every `writes.Forks` event for that path.
A Write or Edit whose result failed is not evidence.
Bound: only these two paths of this run use the rule; every other path under `_lyx`, `.lyx` or the scratch directory stays unverifiable, and the rule clears on evidence (absence, or Master's later write), never on an acknowledgement, which keeps refusal-spec's "a correctness halt clears only on evidence" line true.
Absence clears the audit finding only: a run whose contract files are absent is still in flight.

Every site that classifies a suspect path consults it before `checkSuspectPaths` and treats a cleared contract path as resolved:
- `AcceptPendingAudit` takes the `shuttleengine.Engine`, loads `RunWrites`, and accepts a finding whose only paths are cleared contract paths.
  An uncleared one refuses, naming the path and why ("a fork wrote it after Master's last write"), with `wayForwardSteps("rm <path>", "lyx webster accept-audit")`.
  It also returns whether it accepted on an absent contract file.
- `acceptAuditCmd` passes `c.engine`, and when the accept was on an absent contract file its envelope gains `next`, naming the step that has Master write them again: "re-run `lyx webster run` (in a shed-driven run, re-step the Webster row); a fresh Master resumes from state.json and writes outcome.yaml and summary.md".
  That is the existing re-entry: `Run` archives a stale outcome at entry and a fresh Master with every batch done writes both files, so the run ends only on contract files Master wrote; no new re-entry step is added.
- `RecoverSpawnOrAttach` drops cleared contract paths from a failed batch's `Uncheckable` before its `--fresh` refusal, and an uncleared one gets the delete way forward with `lyx webster recover-batch NN` as the re-run.
- `pendingPathsWayForward`, called from `runExitAuditCrossCheck` and `pendingAuditFindingsError` (both load `RunWrites` from their deps' engine), routes a cleared contract path to the `lyx webster accept-audit` route and an uncleared one to the delete route, rather than to `--fresh`.
  At run exit Master has normally rewritten them already, so `lyx webster accept-audit` clears the finding directly.
- `freshPendingDrop`, through `Run`, applies the same rule: an uncleared contract path refuses with the delete way forward and `lyx webster run --fresh` as the re-run.

The run-exit audit still halts on a fork-contract-write finding (A5); this changes what clears it, not whether it halts.

`refusal-spec.md`: the "audit: fork-contract-write" row's way forward names `lyx webster accept-audit` at run exit once Master rewrote the files, and the delete route otherwise;
a new row "accept-audit: contract file last written by a fork" quotes its way forward;
the recover-batch and `--fresh` rows name the contract-file route.

Tests:
`contractevidence_test.go` (untagged): absent file cleared; Master's succeeded write after the fork's cleared; Master's failed write after the fork's not cleared; the fork's write last not cleared; a non-contract `_lyx` path is not a contract path.
`AcceptPendingAudit` accepts with both files absent, accepts when Master's transcript wrote the file after the fork, and refuses naming `rm <path>` and the reason when the fork's write is last and the file exists;
`acceptaudit_test.go` asserts the `next` field on absent files;
recover-batch and `--fresh` each refuse on an uncleared contract path with the delete way forward and pass on a cleared one.
Grep the tests for the old "cannot be checked" and "nothing the run recorded" texts these sites emit for contract paths, and update each assertion.
Document the evidence rule in `internal/websterengine/doc.go`'s audit section.

**ImpactSummary:** four suspect-path classification sites gain one evidence pre-check for two paths; every other path's classification is unchanged.
