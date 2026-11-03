# Round 1 fixer-report — plan

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  In card 3's Intent, the `docs/overview.md` bullet now also rewrites the orch entry's clause at `docs/overview.md:424` ("ensuring it replaces a `loom-status` strand ... put in the overview window first") to the same wording as the batten entry: the runs strand is added beside such a strand, as the session's first overview window.
- **F2 (LOW), fixed as proposed.**
  - Card 1: `internal/reedengine#isOverview` moved from Uses to Edit, with an Intent line rewording its doc comment to "an overview window, one of the session's first windows".
  - Card 3: Intent lines for `runsStrandAddSpec`'s doc comment and the `runs.go` file header, each calling the window the session's first overview window.
  - Card 2: `internal/reedcli#reedCLI.followCmd` added to Edit and `internal/reedcli/doc.go` added to Uses, with an Intent line rewording the follow help to "else the lowest-index overview window".
    The ImpactSummary now says that outside reedengine only that help text changes.
  - `lym loom validate-plan` answers `ok: true`.
- **F3 (NIT), fixed as proposed.**
  Card 1's `RelaunchingOrReplacingEitherOverviewKeepsTheOrder` bullet now relaunches `pawl` with `AddStrand` `IfAbsent`, which relaunches its stored command `exit`, so the row asserts only the order after the relaunch.
- **Self-check fixes**, below.
  - Card 2: `internal/reedengine#Engine.AttachArgv` added to Edit and to the caller-comment list.
    `TestAttachArgv_SelectsTheRelevantWindow`'s doc comment is reworded to "else the lowest-index overview".
  - Card 1: `internal/reedengine#bindingArgvs` added to Edit, with an Intent line rewording its forward-step target to the lowest-index overview.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal, LOW: `AttachArgv`'s doc comment (`attach.go:43`) and `TestAttachArgv_SelectsTheRelevantWindow`'s doc comment (`attach_test.go:359`) still say "else the overview".
  Fixed as proposed, merged with the fix-affected-set finding on `AttachArgv`.
- fix-affected-set, LOW: `Engine.AttachArgv`'s doc comment is not covered.
  Duplicate of the fix-proposal finding; fixed as proposed.
- fix-affected-set, LOW: `bindingArgvs`'s doc comment (`bindings.go:110`) still calls the first window "the overview".
  Fixed as proposed.

## Disputed

None.

## Deferred

None.
