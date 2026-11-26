```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that this round's fixes cover every member of each finding's set (lens `fix-affected-set`).

**Findings left unchecked:** none.

**no findings**

- **F1:** every `newWiredPairFixture` caller now gets the cleanup. Integration-tier callers call `waitForFixtureProcessesToExit` explicitly, and tmux-tier callers get it through `registerBootstrapTeardown`. The tmux-tier tests that build a hub with `CopyHub` and run `lym` verbs (`TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` and `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps`) also register that teardown. `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver` has neither. It is outside the finding's set: it drives `wire` and `startLLMDriverArm` in-process, runs no verb as a subprocess, and does not use the fixture.
- **F2:** the premise is a single loop. Its doc comment and the summary bullet both match the new behaviour.
- **F3:** the premise is a single test, and it carries the marker.

Tool calls (2 of 6):
1. An awk-filtered grep over `internal/loomcli/*_test.go` mapping each test to its fixtures, verbs and cleanups. My context showed only the fixture's callers, not the tests that build hubs directly.
2. `sed` on `integration_driverbootstrap_test.go` and a grep of `internal/boardengine/spawn.go`. My context held neither the one remaining tmux-tier hub test nor the sync spawn code.
