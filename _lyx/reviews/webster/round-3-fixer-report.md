# Webster-Burler round 3 — fixer-report

All three findings hold: card 7 makes a claude run start only once its events file holds a `SessionStart` line, and the two `loomcli` stub providers printed the ready pane without ever writing that line.
`.lyx/verify/verify-4.log` shows each failure as `outcome "died"` with `no session-start line in the events file` and the ready box on the pane.
The Publish merge-in (`6f7b77362`) touches none of the failing paths.

## Changes

- **F2** — fixed as proposed, commit `76a7b2f92`.
  `claudeengine` gains `SessionStartupPayloadFixture`, the startup `SessionStart` payload line, beside its other hook-name constants in `signals.go`, so the Claude payload shape stays behind `PATTERN-shuttle-provider-seam` the way `IdleInputBoxFixture` does.
  `smoke_driverstrand_test.go` gains `stubRecordSessionStart`, a shell function that takes the `--settings` path from the stub's arguments, finds the events file that settings file's recording hooks append to, and appends the fixture line there before the stub draws its first box.
  `writeStubDriverScript` runs it.
- **F1** — fixed as proposed, commit `acb403c21`.
  `integrationWriteStubDriverScript` runs the same `stubRecordSessionStart`, so the two stubs share it as the review asked.
- **F3** — fixed by F2's change, with no change of its own: `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` takes its provider from `writeStubDriverScript`.

No test was added: none of the findings is a coverage gap, and the three failing tests are the ones the change makes pass.

The review's suggested sweep for other stub providers found none: every other test naming `claudeengine.New()` or a claude fixture is `llm`-tagged against a real `claude`, or drives the `shuttlefake` engine.

## Verification

- `lyx gate test --tags tmux ./internal/loomcli -- -run '<the three tests>' -count=1`: all three pass.
- `lyx gate test --tags tmux ./internal/loomcli -- -count=1`: ok.
- `lyx gate test ./internal/shuttleengine/... ./internal/loomcli -- -count=1`: ok.
- `go vet -tags tmux` and `go vet -tags integration` over the touched packages: clean.
- `lyx loom lint-comments`: ok.

## Disputed

None.

## Deferred

None.
