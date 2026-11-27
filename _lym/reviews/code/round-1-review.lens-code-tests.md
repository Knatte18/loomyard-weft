```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that fails if it breaks.

**Behaviour → test that pins it**

1. `Board.EndBeam` clears the flag, recipe, owner and colour, and keeps the status, kind and member's `project`: `TestEndBeam` "Beam" row plus the field asserts after the loop (`internal/boardengine/beamend_test.go:72,90-99`).
2. `EndBeam` returns false on an ended beam, a plain project, a task or an absent entry: `TestEndBeam` rows (`beamend_test.go:73-76`). "Writes nothing" (the `noWrite` path, with no save or date stamp) is not asserted; only the return value is.
3. `beam: false` upserts cleanly once the beam is ended: `TestEndBeam` (`beamend_test.go:100`).
4. `clearRunPhase` with `--remove` on a beam calls `EndBeam` after `ClearClaim` and sets `BeamEnded`: `TestCancelRun/RemoveOfABeamClearsItsFlagAfterTheClaim`. Order is checked by `callsInOrder` (`internal/battencli/cancel_test.go:1150`).
5. A cancel without `--remove` keeps the flag: `TestCancelRun/BeamWithNoMembersEndsAndStopsItsOrchAfterTheEndKeepingItsFlag`, with `noCalls` containing `EndBeam` and `BeamEnded` false.
6. An `EndBeam` failure refuses, names the re-run and skips `LiftPauses` and `Notify`: `TestCancelRun/FailedBeamFlagClearNamesTheRerun`.
7. An `isBeamProject` error inside `clearRunPhase` gives "could not clear the beam flag": none. The fake's `BeamProject` has no error knob (`cancel_test.go:248`).
8. `refuseNothingLeft` with `--remove` counts a lone beam flag as left, ahead of `TeardownPending`: `TestCancelRun/BeamFlagAloneIsLeftToClearWithRemove`.
9. `refuseNothingLeft` without `--remove` still refuses nothing-left on a lone beam flag: none.
10. A `BeamProject` probe error in `refuseNothingLeft` refuses through the "the board entry" probe refusal: none, same missing knob.
11. `cancelNotice` adds the cleared-flag suffix: the `RemoveOfABeam…` row, by substring `strings.Contains` (`cancel_test.go:171`).
12. `cancelEnvelope` adds the `beam_ended` key: none. No test reads the key; the integration tests check the other keys (`cancel_integration_test.go:166,276,336`) and include no beam.
13. Production `hubCancelDeps.EndBeam` wiring (open the hub board, then `Board.EndBeam`) together with the real `BeamProject` read: none. No integration test cancels a beam.
14. Production `StopBeam` skips a pair whose origin record is absent or unmarked: none. `_lym/landing/summary.md:34` admits this.
15. `checkBeamPayload`'s refusal now names `lym batten cancel <slug> --remove`: the existing `store_test.go:438` row still matches its old substrings, and nothing asserts the new way forward.
16. Help text, `doc.go`, the `cli.go` comment and `docs/overview.md` prose: exempt as static prose.

**Findings**

- **MEDIUM**, design, `internal/battencli/cancelcmd.go:390-397`: the new `StopBeam` skip on an absent or unmarked origin record has no test. This is the fix for the issue's orch-stop warning; removing the `!found || !origin.Beam` guard brings that warning back and every test still passes. Evidence: `cancelcmd.go:394-396`, `summary.md:34`.
- **MEDIUM**, design, `internal/battencli/cancelcmd.go:401-407` and `cancel.go:578-586`: no test runs a real beam `--remove` cancel over a hub board. The issue's own scenario (flag cleared, members untouched, `beam: false` upsert accepted afterwards) is only checked through a fake `EndBeam` whose answer `f.beamProject && f.endBeamErr == nil` (`cancel_test.go:257-260`) is hand-written. A broken wiring, such as the wrong board or slug, or `BeamProject` reading the owners file differently from what `EndBeam` changes, passes every test. Evidence: `cancel_integration_test.go` contains no beam case.
- **MEDIUM**, design, `internal/battencli/cancelcmd.go:169`: the new `beam_ended` key in the JSON output has no test. Dropping it or wiring it to the wrong field fails nothing. Evidence: no `beam_ended` match in any `_test.go`.
- **LOW**, design, `internal/battencli/cancel.go:767-785`: no row pins that a lone beam flag without `--remove` still refuses nothing-left. Moving the beam check above `if !remove` would pass every test. Evidence: `cancel.go:767-775`, and the only lone-flag row sets `remove: true` (`cancel_test.go:177-179`).
- **LOW**, design, `internal/battencli/cancel.go:579-585` and `cancel.go:779-782`: the new `isBeamProject` error branches have no row, because the fake `BeamProject` cannot fail (`cancel_test.go:248`).
- **LOW**, design, `internal/boardengine/project.go:125`: the refusal's new way forward, `lym batten cancel %s --remove`, is not asserted. `store_test.go:438` checks only the substrings that were already there.
- **LOW**, design, `internal/boardengine/beamend_test.go:72-84`: the no-op rows assert only `false`. The "writes nothing" contract (`beamend.go:15`, `noWrite`) is unchecked, for example that a plain project's `revised` date is unchanged.
- **NIT**, design, `internal/battencli/cancel_test.go:171`: the notice is asserted by substring rather than exact equality, which the testing rules prefer.

Outside this lens: the new `EndBeam` fake in `cancelFake` stands in for `boardengine`'s behaviour, which may conflict with `PATTERN-test-stand-ins` ("No change adds a stand-in this rule forbids"). That is for the handler or the affected-set lens.

**Tool calls**

1. grep for `wantCalls`, `noCalls` and `beam_ended` across the battencli tests: needed to see how the table asserts calls and whether any test reads the envelope key.
2. grep inside `cancel_test.go` for the `callsInOrder`, `noCalls` and marker checks and the beam rows: needed to confirm ordering is asserted and to see which beam rows exist.
3. grep for the beam-payload refusal text in tests, for `cancelEnvelope` tests and for beam cases in the integration test: needed to check coverage of the changed message, the envelope and the production wiring.
