# Move Webster's batch machinery into a shared runplan package

Webster's batch machinery and run-plan state move out of `internal/websterengine` into a new package, `internal/runplan`.
The Weaver implementer can then reuse them, and two copies of the post-batch checks cannot drift apart.
Webster's behaviour does not change, and its tests keep their assertions.
The test edits only follow the moved names.

## What moves

`internal/runplan` now holds:

- the run-plan state: plan fingerprint, plan file hashes, overview frame hash, recorded partition and batch records;
- the batch report contract and its parser, with a new optional `journal` field that Webster's forks may leave empty;
- the plan fingerprint, its re-baseline, and the dispatch re-resolution with its plan-conflict error and record;
- the post-batch checks: done checks, handle binding, the scope guard, drift repair, and renames that rewrite pending cards;
- the suspect-path checks, the failure digest and card gate rendering;
- the bounded wait for a batch report, the recovery strand stop and the verify-failure record.

`websterengine.State` embeds the run-plan state beside its Master fields.
Fields only Webster fills stay in `websterengine` or are optional in the shared batch record.
`websterengine` keeps aliases for the moved names, so its importers compile unchanged.
The fork transcript audits stay in `websterengine`.

Importers and tests in `webstercli`, `loomshed`, `loomcli`, `shedadapters`, `failuretrace`, `hubgeom`, `standalonegeom` and `tools/tokencount` follow the moved names.
`docs/overview.md` lists the new package.
Two `testOnlyCodeAllowed` entries in `cmd/lym/testonlycode_linux_test.go` are dropped, because `Fingerprint` and `WriteReport` are no longer `websterengine` exports.

## Design decisions

- **Geometry.**
  Each shared piece takes its paths from a `runplan.Geometry` the caller passes.
  It names the implementer's run, reports, prompts and scratch directories, and the scratch directory holding the state-mutation lease.
  It also carries the `Git` surface and the code index the checks read through.
  Under Weaver these directories name Weaver's own, so its lock and lease sit under `.lym/weaver/`.
- **Way forward from the caller.**
  A shared refusal takes its way-forward text from the caller.
  Webster passes its rebaseline and restore-plan wording, and Weaver passes its own.
  Two texts keep Webster's names because readers pin them: the begin-batch header of the plan-conflict record and the handle-bind log line.
- **Recorded partition.**
  The post-batch check takes the recorded partition already mapped onto the current plan, the mapping Webster's verbs use.
  The later-reference check therefore sees the pending cards, and only the run's last batch reads as last.

## To check

- Run the full test suite, including the integration tier.
  I did not run it for this description.
  No regression check ran for this task.
