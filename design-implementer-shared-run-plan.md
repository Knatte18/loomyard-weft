# Move Webster's batch machinery and run-plan state into a package both implementers import

`implementer-shared-run-plan` · Task · enhancement, webster, weaver · running · Worktree-Create

Decision record: [Sketch](decision-implementer-shared-run-plan.md)

Filed from the Sketch `weaver`.

Takes: Webster's batch machinery as it stands in internal/websterengine: report.go, digest.go, fingerprint.go, the plan-conflict refusal of beginbatch.go and conflictrecord.go, postBatchChecks in recordbatch.go, suspect.go, cardgate.go, awaitbatch.go and strand.go, verifygatereport.go and verifyparse.go, and the run-plan fields of state.go's State and BatchState
Delivers: a new package internal/runplan holding the run-plan state (plan fingerprint, plan file hashes, overview frame hash, recorded partition, batch records) that websterengine.State embeds beside its Master fields, with every Webster-only batch-record field kept in websterengine or optional; the batch report contract and parser with an optional journal field; the plan fingerprint and plan-conflict refusal; the post-batch checks; the suspect-path checks; the failure digest; card gate rendering; strand await; and the verify-failure record. Each shared piece takes the implementer's run, reports, prompts and scratch directories and its state-mutation lease from a geometry the caller passes, and each shared refusal takes its way-forward text from the caller, so Webster passes its own and its behaviour and existing tests stay unchanged
Exports: package internal/runplan: the run-plan state type and batch-record type both implementers' state files embed, the report type with Journal and its parser, the post-batch check entry point taking a recorded partition mapped onto the current plan, the suspect check, the digest, the card gate renderer, the strand await, the verify-failure record, the plan-conflict error and record, and the implementer geometry type; websterengine keeps aliases for the names it moved so its importers compile unchanged

The part changes these packages:

- internal/runplan (about 1600 changed lines)
- internal/websterengine (about 900 changed lines)
- internal/webstercli (about 200 changed lines)
- internal/loomshed (about 80 changed lines)
- internal/loomcli (about 80 changed lines)
- internal/shedadapters (about 60 changed lines)
- internal/failuretrace (about 30 changed lines)
- internal/hubgeom (about 20 changed lines)
- internal/standalonegeom (about 20 changed lines)
- tools/tokencount (about 20 changed lines)
