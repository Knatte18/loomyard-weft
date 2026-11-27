```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: for each output, message, key and behaviour the diff changes, I found what reads or describes it and checked whether the diff leaves any of it wrong.

**Per change, the set found and whether it is covered:**
- **`checkBeamPayload` refusal text** (`internal/boardengine/project.go:68`): `internal/boardengine/store_test.go:438` asserts the substrings `"beamer"`, "stays set for the beam's life" and "lym beam land beamer", and all three still match. The new clause is asserted nowhere, which is fine because it is static prose. Covered.
- **The new envelope key `beam_ended`** (`cancelcmd.go:169`): the integration tests at `cancel_integration_test.go:166,187,219,276,336` assert the other keys but never `beam_ended`. The help text's `Output:` paragraph lists the envelope fields but not `beam_ended`. Not covered (F2).
- **The cancel help text** (`cancelcmd.go:66-76`): the sentence "It skips every wait a teardown makes for a working driver." moved out of the user help into the package doc (`cli.go:24`), which users don't see. No test asserts it. It is a user-facing statement that was dropped (F3).
- **`StopBeam` now skips an unmarked pair**: its only reader is `stopBeamOrch`. The `cancelDeps` field comment (`cancel.go:131`) was updated. Covered.
- **The `EndBeam` seam and the `refuseNothingLeft` beam branch**: `hubCancelDeps` is the only production constructor of `cancelDeps` (`cancelcmd.go:199`). Its callers are the CLI at `cancelcmd.go:119` and the exported `Cancel` at `cancelcmd.go:141`. `Cancel` is called by `internal/fronttask/hub.go:67`, where `remove` is true but the slug is a front task, never a beam, so nothing changes there. Covered.
- **The beam-flag lifecycle in docs**: `docs/overview.md:1737` was added. The `cli.go` package doc lines 44-45 were added. The `boardengine/doc.go` line was added. `internal/boardcli/cli.go:592` says only the "lym beam" verbs set the flag; batten cancel only clears it, so that line is still accurate. Covered.
- **The test fake `cancelFake`'s new `EndBeam` seam** (`cancel_test.go:137-140`), checked against PATTERN-test-stand-ins: not compliant (F1).
- **Commit trailers**: `16b2aba88` carries a `Root-Cause:` trailer, and it names the defect the diff removes (`clearRunPhase` never cleared the flag). No `Pattern-Departure:` or `Loosened-Check:` trailer appears in the range, and none is needed: no assertion was loosened. The rename of the existing row (`BeamWithNoMembers…KeepingItsFlag`) only adds assertions. Covered.
- **Declared PATTERN changes**: the directive declares none, so there is nothing to check.

**Findings:**

**F1. BLOCKING, design.** Location: `internal/battencli/cancel_test.go:137-140`. The diff adds a new hand-written stand-in for another repository module. The fake `EndBeam` seam answers `f.beamProject && f.endBeamErr == nil` and invents the board error "board is locked" (row `FailedBeamFlagClearNamesTheRerun`). In production that seam is filled by `boardengine.Board.EndBeam` (`cancelcmd.go:245-251`), so the fake simulates `boardengine`'s state and output. PATTERN-test-stand-ins says "An interface the module under test declares does not make its fake the module's own: when another module fills the seam in production, the fake simulates that module", "An error of another module is provoked in the real module", and "No change adds a stand-in this rule forbids". Evidence: `pattern/PATTERN-test-stand-ins.md` (sections "A repository module has no stand-in" and "Standing fakes"). `cancelFake` is also not listed in the standing-fakes inventory: a grep of `docs/research/no-fake-other-modules.md` for battencli or cancel found nothing. Fix: drive the flag clear through the real board, for example in `cancel_integration_test.go`, rather than through a fake seam answer.

**F2. LOW, consistency.** Location: `internal/battencli/cancelcmd.go:77-78` (the help's `Output:` paragraph). The paragraph lists the envelope's fields ("whether the pair and record were removed or kept, the claim cleared and the push spawned") but not the new `beam_ended` key that `cancelEnvelope` now emits (`cancelcmd.go:169`). No integration test asserts the key either (`cancel_integration_test.go:166,336`).

**F3. LOW, consistency.** Location: `internal/battencli/cancelcmd.go:68-69`. Shortening the help to fit the length cap (commit `dddaa2916`) dropped "It skips every wait a teardown makes for a working driver." from the user-facing `--help`. That sentence now lives only in the package doc at `internal/battencli/cli.go:24`, which a CLI user never sees. The summary does not mention the removal. A shorter rewording elsewhere would have kept it.

Tool calls:
1. Grepped for the changed refusal text, the help sentence and the envelope keys across code and docs, to find who reads or asserts them. My context held only their definitions.
2. Grepped for files and docs that describe the beam flag, to find lifecycle docs the change could leave stale. They were not in my context.
3. Read the beam-flag lines in `beamcli`/`boardcli`/`hubgeom` docs and `docs/overview.md`, to check that their flag-lifecycle wording is still correct.
4. Grepped for constructors of `cancelDeps` and the stand-in inventory, to check for other production deps that might lack `EndBeam`, and whether `cancelFake` is a listed standing fake.
5. Grepped the inventory for battencli and read `cancelcmd.go:135-150`, to confirm `cancelFake` is not listed and to see the exported `Cancel` caller path.
6. Grepped for callers of `battencli.Cancel(`, to find out-of-package callers whose behaviour the beam-flag clear changes.
