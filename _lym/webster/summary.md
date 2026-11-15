# One run FSM over tracked records, with cancel and pause intents

All six batches of the card list recorded `status: done`, one card per batch in the digests.
No batch needed recovery, and no digest carried a rename, a departure or a warning.

## What was built

- **01-shed-run-intents** (head `f7a5ceef`): the cancel and pause intent fields and the cancelled state in the batten record.
  Writers under the status lock in `internal/shedrun`.
  The new leaf package `internal/runfsm` with `Derive`, the act rule, `Judge` and `TransitionState`.
  Three reads of origin's fetched refs in `internal/fabricengine`.
- **05-pairteardown-read-run** (head `6638ae10`): `pairteardown.ReadRun` as the one reader of a run's FSM inputs.
  The board's run status is written from the FSM state.
  The cancel records its intent and finish, keeps or deletes the record, and asks the act rule.
- **08-cancel-readers-intent** (head `cefc3b5e`): every cancel-marker reader reads the record's intent, and the marker file is gone.
  Cleanup and teardown delete a cancelled record whose pair is gone.
  A hand `lym batten run` and `lym batten step` ask the act rule.
- **11-dobby-pass-act** (head `1edab1f1`): the dobby's pass and `lym loom resume` ask the act rule.
  `boardengine.PullBolt` is shared with `tendercli`.
  `lym tender pause` and `resume` write and clear the pause intent, and refuse an owner elsewhere.
- **15-pr-record-tracked** (head `7a65ae99`): the pull-request record and the fixer attempt counts live in the tracked records, and each record write commits and pushes.
- **17-lym-files-pattern** (head `ad20420b`): `PATTERN-lym-files` now says `.lym` holds only caches.
  A fix trimmed two batten help texts that earlier batches had pushed past the size cap.

## Deviations and departures

Every digest listed deviations, files changed outside the batch's own targets: the test-only-code allowlist, shared test kits and fixtures, and the touched call sites.
They are informational, and the scope guard failed no batch.
No digest listed a `Pattern-Departure:` trailer.

## Verify gate findings

The webster gate failed on two tests:

- `internal/battencli.TestWire_EverySeamFilled`
- `internal/loomcli.TestSmokeBootstrapLifecycle/malformed_status_proceeds_to_the_success_envelope_and_logs_why`

The findings file named cards 05, 08, 15 and 17 as touching a failing package.
One fixer fork handled both findings, and the gate verifies again after this turn.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `d859834728256df8f8b6b7e7b0dd3b5de8cbf72d` fix: let an undecodable run record reach the driver's own read, and list batten's unused CommitRecord seam
