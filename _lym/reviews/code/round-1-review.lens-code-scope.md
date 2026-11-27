```yaml
lens: code-scope
outcome: reported
```

Scope review of the diff against the board entry's goal points, mapping each change to the point it builds. The decision record has no `Added after Discussion` entries.

**Decisions (goal points of the board entry):**
- G1 — `batten cancel --remove` of a beam's pawl run ends the beam's life and clears the project's beam flag.
- G2 — the result is a plain project whose members are untouched.
- G3 — the cancel's false warning "the beam's orch was not stopped … is neither" (the entry reports it as a defect).
- G4 — alternative: a verb such as `lym beam cancel <slug>`. The entry offers it only as an alternative to G1, so not building it is no gap.

**Changes → decision:**
- C1 `internal/boardengine/beamend.go` `Board.EndBeam`, plus `beamend_test.go` and the `doc.go` line → G1, G2 (keeps members' `project` field).
- C2 `cancel.go` `clearRunPhase` calls `EndBeam` under `remove` → G1.
- C3 `cancel.go` `refuseNothingLeft` counts a beam flag as left under `remove` → G1, so a beam already stuck this way is finished by a re-run.
- C4 `cancelReport.BeamEnded`, the envelope's `beam_ended`, the `cancelNotice` suffix and the `cancelDeps` docs → forced by C2.
- C5 `cancelcmd.go` `StopBeam` skips a pair whose origin record has no beam mark → G3.
- C6 `project.go` `checkBeamPayload` refusal now names the cancel with `--remove` → forced by C1/C2, since the old text named `beam land` as the only end.
- C7 `docs/overview.md`, the cancel help text and the `cli.go` package doc → forced by C2/C3. Part of it moves "It skips every wait a teardown makes for a working driver." out of the help into the package doc, to stay under the help length cap.

**Decision → changes:**
- G1: C1, C2, C3 (C4, C6, C7 forced).
- G2: built only for a beam with no open members (C1 keeps members as they are). See finding 1.
- G3: C5.
- G4: none (alternative, not required).

**Findings:**

1. BLOCKING · scope · `internal/battencli/cancel.go:483` (`refuseBeamCancel`), with `internal/battencli/beamland.go:113-135` and `internal/boardengine/parentbeam.go:32`
   The entry's scenario, a beam with members that never ran, still cannot reach the end G2 asks for, so G2 is not built for the case the issue reports.
   - A draft member with no origin record still reads the beam as its parent through `ParentBeam`, so `beamMembers` lists it.
   - `BeamLandRefusal` then refuses the cancel and tells the user to move each member's `project` off the beam first. That is the hand change to the members the entry says to avoid.
   - `EndBeam` (C2) only runs once the beam has no members, so "a plain project whose members are untouched" holds only when there are no members.
   - The `TestCancelRun` row `BeamWithMembersRefusesNamingThemWithNothingTakenOrEnded` (`internal/battencli/cancel_test.go:943-950`) confirms the refusal still stands with members.
   - The new help line "its members untouched" (`cancelcmd.go`) and `cli.go`'s "its members are untouched" promise an outcome that refusal rules out.

2. LOW · scope · `internal/battencli/cancelcmd.go:69` (help text)
   The help-length fix drops the user-facing statement "It skips every wait a teardown makes for a working driver." from `lym batten cancel --help` and moves it into the `cli.go` package doc.
   - No goal point asks for that.
   - Only the length cap the C7 addition hit forces it, and the cap could have been met by tightening the new text instead.
   - Evidence: the cancelcmd.go hunk at diff `002-internal-battencli.diff:211`, and `cli.go:263` in the same diff.

Tool calls:
1. grep of `cancel_test.go` for the beam-member rows, to confirm a beam with members still refuses; my context held the production refusal but not the test rows.
