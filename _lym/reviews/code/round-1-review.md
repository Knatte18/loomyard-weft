---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/battencli/cancel.go#refuseBeamCancel
    summary: The board entry's scenario, a beam whose members never ran, still refuses the cancel and asks for each member to be moved off the beam, so the plain project with its members untouched is never reached.
    fix: "In refuseBeamCancel, when remove is set, stop refusing for a member that has neither a pair (HasPair) nor a run status (Running): such a member keeps its project field, and once EndBeam clears the flag ParentBeam reads the prime as its parent. Keep the refusal for a member with a pair or a run in flight, for a pair forked from the beam, and for every member of a cancel without --remove, which keeps the flag. Thread remove into refuseBeamCancel from endRunPhase. Both refusal branches need the change: the unheld branch currently refuses on any non-nil LandRefusal, so read the blockers there too and refuse only when one of the kept kinds is present. Add a TestCancelRun row: a beam with --remove whose only member is a draft with no pair and no run status ends the run and calls EndBeam. Change the existing BeamWithMembersRefuses... row so it keeps a refusing member with a pair or a run status. Update `_lym/landing/summary.md` so its claim that an affected hub needs no hand edit covers members."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/battencli/cancel_test.go:137
    summary: "The new cancelFake EndBeam seam is a hand-written stand-in for boardengine: it answers beamProject && endBeamErr == nil and fabricates a board error. PATTERN-test-stand-ins forbids any change from adding one. No test drives a real beam --remove cancel over a hub board, so the production EndBeam wiring and the new beam_ended envelope key go unchecked."
    fix: "Add a TestCancelIntegration_ test in internal/battencli/cancel_integration_test.go. It seeds a hub (hubforge) with a beam project that has no members and an ended pawl run, runs the real `lym batten cancel <slug> --remove`, and asserts four things: the board entry's beam is false, its recipe, owner_machine and beam_color are empty, the envelope's beam_ended is true, and an upsert of beam false is accepted. Then drop the fabricated answer from cancelFake. Its EndBeam either returns a fixed false while recording the call, so the table keeps only the order assertion with no board outcome, or is removed from the rows that assert a board outcome (RemoveOfABeamClearsItsFlagAfterTheClaim, BeamFlagAloneIsLeftToClearWithRemove). Remove the FailedBeamFlagClearNamesTheRerun row's invented \"board is locked\" error, or provoke a real failure in the integration test through the board's state, for example an unreadable board directory via permkit.Chmod."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/beamend.go:19
    summary: EndBeam clears a beam's owner_machine outside SetOwners, which boardengine's doc names as the only owner writer, the one that pulls first, writes one commit and pushes. The diff leaves that invariant's text false.
    fix: "Choose one of two fixes. (a) Clear the owner through the owner path: have EndBeam's owner change go through the SetOwners machinery (pull, one commit, push), or call it from the battencli EndBeam seam. (b) Keep the write and fix the invariant: amend the `SetOwners is the only writer of an owner` paragraph in internal/boardengine/doc.go to name Board.EndBeam as the one other writer, which clears a beam's owner when its life ends with no landing, and state there why the ordinary write and detached sync suffice (no other machine owns an entry this machine just cancelled). Option (b) is the smaller change."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/cancelcmd.go:396
    summary: The StopBeam skip on an absent or unmarked origin record, which is the fix for the orch-stop warning in the issue, has no test. Deleting the `!found || !origin.Beam` guard brings the warning back and every test still passes.
    fix: "Add a test in internal/battencli that drives the production StopBeam seam from hubCancelDeps over a hubforge hub whose pair for the slug has an origin record without the beam mark (as fabric writes it before Beam-Up). Assert that it returns nil and that the cancel's Warnings carry no `the beam's orch was not stopped` entry. It can be a row of the integration test F2 adds. Then drop the `no test pins it` sentence from `_lym/landing/summary.md`."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:38
    summary: "The change description calls the merge danger two-way with blast radius batten, but the diff adds a boardengine write that erases a beam's recipe, owner_machine and beam_color and pushes the change outward through the board's sync. That is data deletion and an outward push, and the radius leaves out boardengine."
    fix: "In `_lym/landing/summary.md`'s `## Merge danger`, set the door to one-way, or keep two-way only with a stated reason why the erased fields are recoverable. Widen the blast radius to `batten, boardengine (board entries of cancelled beams)`."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battencli/cancelcmd.go:68
    summary: "The help-length fix moved `It skips every wait a teardown makes for a working driver.` out of `lym batten cancel --help` into the cli.go package doc, where a CLI user never sees it. The change description does not mention the removal."
    fix: "Restore the sentence to the cancel help text in internal/battencli/cancelcmd.go, after the refusal list, and meet the length cap by tightening the new beam paragraph instead, for example by merging its last line into the sentence before. Remove the duplicated line from internal/battencli/cli.go:24 if the package doc no longer needs it."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battencli/cancelcmd.go:77
    summary: The help's Output paragraph lists the envelope's fields but not the new beam_ended key the envelope now emits.
    fix: "In the help text's `Output:` paragraph in internal/battencli/cancelcmd.go, add the cleared beam flag beside the claim cleared, for example `the claim cleared, the beam flag cleared (beam_ended) and the push spawned`, within the length cap."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/battencli/cancel_test.go:177
    summary: "The new branches lack rows. Nothing pins that a lone beam flag without --remove still refuses as nothing left, and neither new isBeamProject error branch, in refuseNothingLeft or in clearRunPhase, is reached, because the fake BeamProject cannot fail."
    fix: "In TestCancelRun, add a row with remove false, beamProject true and nothing else left (as in BeamFlagAloneIsLeftToClearWithRemove), wanting the nothing-left refusal (errors.Is ErrNothingLeft) and no EndBeam call. If a BeamProject error knob is added, add one row each for the refuseNothingLeft probe refusal (`the board entry`) and the clearRunPhase refusal naming the --remove re-run. Follow F2's stand-in rule for any knob added: provoke the error through real board state in the integration test instead if a fake knob would fabricate it."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battencli/cancel.go:583
    summary: An EndBeam failure returns after RemoveRecords has committed the records deletion and before PushRecords runs. The re-run makes no new records commit, so pushRecords stays false and this cancel never pushes that commit itself.
    fix: "In clearRunPhase, when RemoveRecords reports no commit on a run with remove set but the records side has an unpushed commit, still push. Alternatively, move the EndBeam call after the PushRecords block so a failed flag clear never skips the owed push, keeping its refusal naming the re-run. The second option is the smaller change."
---

### [BLOCKING] A beam with members still cannot be cancelled into a plain project with its members untouched

**Location:** `internal/battencli/cancel.go#refuseBeamCancel`, with `internal/battencli/beamland.go:113-135` and `internal/boardengine/parentbeam.go:32`

**Issue:** The board entry's beam had members that never ran, and the entry expects "a plain project whose members are untouched".
A draft member with no origin record reads the beam as its parent through `ParentBeam`, so `beamMembers` lists it and `refuseBeamCancel` refuses with the advice to move each member's `project` off the beam.
That is the hand edit of the members the entry wants to avoid, and `EndBeam` is only reached for a beam with no members.
The new help line "its members untouched" and the package doc's "its members are untouched" promise an outcome the refusal rules out.
The change description also claims that an affected hub needs no hand edit.

### [BLOCKING] The cancel table adds a stand-in for boardengine, and no test drives the real beam cancel

**Location:** `internal/battencli/cancel_test.go:137`

**Issue:** `cancelFake.EndBeam` answers `f.beamProject && f.endBeamErr == nil` and invents the error "board is locked".
In production `boardengine.Board.EndBeam` fills the seam (`cancelcmd.go:245-251`), so the fake simulates another repository module's state and errors.
`PATTERN-test-stand-ins` says the fake of a seam another module fills simulates that module, that such an error is provoked in the real module, and that no change adds a stand-in the rule forbids.
`cancelFake` is not in the standing-fakes inventory.
Because of this, no test runs a beam `--remove` cancel over a real hub board.
The production wiring (`hubCancelDeps.EndBeam` with the owners-file `BeamProject` read) and the new `beam_ended` envelope key are unchecked.

### [MEDIUM] EndBeam writes a beam's owner outside the only owner writer

**Location:** `internal/boardengine/beamend.go:19`

**Issue:** `internal/boardengine/doc.go` says "SetOwners is the only writer of an owner", and that writer pulls the board first, writes one commit and pushes it.
`EndBeam` clears `OwnerMachine` through the ordinary write and its detached sync.
That makes a second owner writer, and the package doc's invariant is now false.

### [MEDIUM] The orch-stop skip has no test

**Location:** `internal/battencli/cancelcmd.go:396`

**Issue:** The production `StopBeam` skip on an absent or unmarked origin record is the diff's fix for the warning the issue reports.
No test reaches it, as the change description concedes, so removing the guard passes every test.

### [MEDIUM] The merge-danger call understates a board-data deletion

**Location:** `_lym/landing/summary.md:38`

**Issue:** The change description calls the door `two-way` with blast radius `batten`.
The diff adds a board write in `boardengine` that erases a beam's `recipe`, `owner_machine` and `beam_color`.
The board's sync then pushes that write outward.
Under the rubric, a two-way call on a diff that deletes data and pushes it outward is a finding, and the stated blast radius leaves out boardengine.

### [LOW] The cancel help lost a statement users need

**Location:** `internal/battencli/cancelcmd.go:68`

**Issue:** To meet the help length cap, "It skips every wait a teardown makes for a working driver." moved from `--help` into the `cli.go` package doc, which a CLI user never reads.
No goal point asks for that, and the change description does not mention it.

### [LOW] The help's Output paragraph omits beam_ended

**Location:** `internal/battencli/cancelcmd.go:77`

**Issue:** The paragraph lists the envelope's fields: the pair and record removed or kept, the claim cleared and the push spawned.
It does not list the new `beam_ended` key.

### [LOW] The new refuseNothingLeft and clearRunPhase branches lack rows

**Location:** `internal/battencli/cancel_test.go:177`

**Issue:** Only the `--remove` case of a lone beam flag has a row.
Moving the beam check above `if !remove` would pass every test.
Neither new `isBeamProject` error branch is reached, since the fake `BeamProject` cannot fail.

### [LOW] A failed flag clear can strand the records-removal commit unpushed by the cancel

**Location:** `internal/battencli/cancel.go:583`

**Issue:** `RemoveRecords` commits, then `EndBeam` fails and the cancel returns before `PushRecords`.
The re-run's `RemoveRecords` makes no new commit, so `pushRecords` stays false and this cancel never pushes that commit itself.
The diff adds this failure point after the commit, beside the one `ClearClaim` already has.

## Rejected

- The new way forward in `checkBeamPayload`'s refusal (`project.go:125`) is unasserted (lens:code-tests): the existing `store_test.go:438` row still reaches the refusal, which `PATTERN-refusal-way-forward` asks for, and pinning the added clause would only assert static prose.
- `TestEndBeam`'s no-op rows assert only the return value, not that nothing was written (lens:code-tests): skipping the write after a `noWrite` is shared `boardCriticalSection` machinery, not behaviour this diff adds.
- The notice is asserted by substring (lens:code-tests, NIT): the notice is prose an orch reads, and exact equality would pin its wording, which the test-economy rules count against a test.
