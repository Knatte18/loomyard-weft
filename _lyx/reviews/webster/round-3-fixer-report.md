# Webster round 3 — fixer report

## Fixed

### F1 — stub providers in loomcli never draw an idle input box

Validated: `IdleSession` needs a caret line between two rules with a blank interior, and both stubs printed only `ReadyFooterFixture`.
`.lyx/verify/verify.log` shows the skill-load send failing with "the pane is not idle".

Commit `00b9be707`:

- `internal/shuttleengine/claudeengine/startup.go`: added the exported `IdleInputBoxFixture` beside `ReadyFooterFixture`.
  It holds a rule, an empty caret line, a rule and the `? for shortcuts` footer.
- One row each in the existing `TestIdleSession` (idle) and `TestStartup_Classification` (ready) tables pins the fixture to both classifiers.
  These are rows in existing tests, not new tests.
- `internal/loomcli/integration_driverbootstrap_test.go` and `internal/loomcli/smoke_driverstrand_test.go`: both stubs print the fixture at start and again after each line they read.
  The smoke stub now also skips a leading `/color` line, as the integration stub already did.
  Without that skip, its Stop event would land before the skill-load send and the load would wait out its timeout.
  The doc comments now describe the new behavior.
- No other stub stands in for `claude`: `grep` finds `ReadyFooterFixture` and `stub-claude` only in these two files and in claudeengine's own tests.

Verification, scoped to the touched packages as the orch directed:
`go test -count=1 -tags 'integration tmux' ./internal/loomcli/ ./internal/shuttleengine/claudeengine/` passed.
That run includes `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`, `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` and `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand`.
`lyx loom lint-comments` passes.
I did not run the full suite.

## Disputed

None.

## Deferred

None.
