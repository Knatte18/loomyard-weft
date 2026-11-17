# Transition push and takeover hand-over: batches 06 to 12

Batches 01, 03 and 04 were already done before this session.
This session ran batches 06, 07, 08, 09, 11 and 12, each `done` on the first `record-batch`.
No batch needed recovery, no digest carried departures or renames, and no digest carried warnings.

## Batches

- **06 run-hand-over** (head `0cc5ae4a`): fabric's hand-over git steps, pairteardown's `PushRun` and `HandOver`, and the commit-records hand-over.
  Deviations: `internal/fabriccli/weft_verbs.go`, `internal/fabricengine/destroy.go`, `internal/fabricengine/handover.go`, `internal/fabricengine/handover_integration_test.go`, `internal/fabricengine/ownerarchive.go`, `internal/pairteardown/handover_integration_test.go`.
- **07 read-run-owed-and-tip** (head `12d1990a`): `ReadRun` fills the owed push and the code-tip wait from origin, and the owner's tip guard is in.
  Deviations: `internal/fabricengine/codetip_integration_test.go`, `internal/fabricengine/handover.go`, `internal/fabricengine/originreads.go`.
- **08 transition-sync-push** (head `a8114e14`): `statuscommit` pushes synchronously under a gate admission and retries an owed push.
  Deviations: `internal/battencli/commitstatus.go`, `internal/fabricengine/pushanchored.go`, `internal/fabricengine/pushowed_test.go`, `internal/statuscommit/statuscommit.go`, `internal/statuscommit/statuscommit_test.go`.
- **09 intent-sync-push** (head `b0cedb6e`, covering cards 9 and 10): tender pause and resume, batten cancel and the pull-request record push synchronously, and the board `done` is written synchronously.
  Deviations: `internal/battencli/cancel_test.go`, `internal/tendercli/runintents.go`.
- **11 finalize-push-order** (head `392c8f64`): Finalize pushes the parent, then its records, then the board `done`, reading the owner before each.
  Deviations: `internal/landingshed/commitstatus_test.go`, `internal/landingshed/finalize.go`, `internal/landingshed/finalize_owner.go`, `internal/landingshed/finalize_pr_test.go`, `internal/landingshed/finalize_test.go`, `internal/loomcli/landingdeps.go`.
- **12 code-tip-grace** (head `a0642d13`, covering cards 12, 13 and 14): the owner's dobby gives up a missing code tip after the grace, `lym beam status` shows the owed push, and `PATTERN-run-fsm` is added with `PATTERN-pair-remote` changed.
  Deviations: `internal/battencli/codetipsettle.go`, `internal/battencli/codetipsettle_test.go`, `internal/battencli/dobbyreads.go`, `internal/battencli/overview.go`, `internal/battencli/overview_test.go`, `internal/fabricengine/codetipsettle.go`, `internal/fabricengine/codetipsettle_integration_test.go`.

## Scope guard and verification

The scope guard listed only the deviations above, all informational.
The batch gates ran tier0 and tier1; tier2 and tier3 did not run.
The plan-level `webster` gate runs after this turn ends.
