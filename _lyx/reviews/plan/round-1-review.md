---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/08-timing-spans-phases.md (Intent, Phases paragraph and the TestComputeTiming every-table-entry row)
    summary: Card 8 places batten rows only before the first Run-Shed entry (setup) or after the last one (land), so a batten row between two Run-Shed entries gets no phase and falls to other with a finding, while its own test row claims such rows count under their phase and the decision record puts every batten row after Run-Shed in land
    fix: In card 8's Phases paragraph, replace "`land` every batten row after batten's last `Run-Shed` entry" with "`land` every batten row other than `Run-Shed` after batten's first `Run-Shed` entry", so the batten rows split at the first `Run-Shed` entry into `setup` before it and `land` after it, as the decision record's table states. In the `TestComputeTiming` row "a landed run whose histories hold one row of every table entry, batten rows before, between and after its `Run-Shed` entries included", state that the between row asserts `land`.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-history-caller-field.md (Intent, Resume paragraph, `Shed.stepLocked` and `TestStep_ResumeFromHaltedStates`)
    summary: Card 5 has the plain step's resume write record ResumedBy when it resumes from paused, while RecordResumer, the until-stop path for the same resume, leaves a paused run unmarked; the decision record scopes the label to resumes of a halted run, so the two paths disagree and the plain-step path goes beyond the record
    fix: In card 5's Resume paragraph, make `Shed.stepLocked`'s resume write set `PendingBy` to `ResumedBy` only when the status it read is blocked, failed or awaiting, `ResumedBy` is non-empty and no label is pending, matching `RecordResumer`. In the `TestStep_ResumeFromHaltedStates` sentence, drop "and for paused too, since a resume from a pause is one" and add a row resuming from paused with `ResumedBy` `orch` that asserts the appended entry carries no `by` and `PendingBy` stays empty.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-caller-label-wiring.md (Create `plan:internal/battencli#TestBattenAutomaticMoves_LabelShed`; Tests paragraph; test-budget sentence)
    summary: Card 6 plans a new top-level test, TestBattenAutomaticMoves_LabelShed, for the wiring of batten's driver seams, which the existing internal/battencli#TestBattenDriverDeps in driverdeps_test.go already covers by calling driverDepsFor and asserting each seam; PATTERN-test-economy puts the new assertions in that test
    fix: "In card 6, remove the Create bullet `plan:internal/battencli#TestBattenAutomaticMoves_LabelShed`, add `internal/battencli#TestBattenDriverDeps` to the Edit list, and remove `cmd/lyx/testdata/test-budget.yaml` from the Edit list. Rewrite the TestBattenAutomaticMoves_LabelShed sentence as: `TestBattenDriverDeps` in `internal/battencli/driverdeps_test.go` gains the assertions that `childGotoArgv(\"Merge-In\")` is `loom goto --to Merge-In --by shed`, and that the `deps.Goto` and `deps.Step` seams `driverDepsFor` returns, over the receiver's status seeded blocked with `untilStop` nil, write a goto entry with `By` `shed` and leave `PendingBy` `shed`. Replace the budget sentence with: `internal/battencli` keeps its row of 83 and `internal/loomcli` stays within its row, so the budget file is not touched. Checked with validate-plan --overlay: no new blocking finding."
---

### [MEDIUM] Batten rows between Run-Shed entries have no phase

**Location:** `_lyx/plan/08-timing-spans-phases.md`, Intent, Phases paragraph.

**Issue:** The decision record's phase table puts "batten rows before `Run-Shed`" in `setup` and "batten rows after `Run-Shed`" in `land`.
Card 8 narrows this to "every batten row before batten's first `Run-Shed` entry" for `setup` and "every batten row after batten's last `Run-Shed` entry" for `land`.
Under those rules, a batten row between two `Run-Shed` entries is in neither set, so it falls to "Any other row counts under `other` and adds a finding naming it".
`Run-Shed` writes one budget-exempt state-change entry each time batten observes the child (`battenshed.ClassifyRecords` reads them).
So a `Land` entry that halted, followed by a goto back to `Run-Shed`, lands between `Run-Shed` entries and would be counted under `other` with a finding.
The card's own test row asserts the opposite: "batten rows before, between and after its `Run-Shed` entries included, each counted under its phase".
That row names a phase the rules never assign.

### [MEDIUM] Plain step and until-stop step disagree on a resume from paused

**Location:** `_lyx/plan/05-history-caller-field.md`, Intent, Resume paragraph.

**Issue:** `Shed.stepLocked`'s resume write runs whenever the status it read is not running, and the card's test asserts that it records the label "for paused too, since a resume from a pause is one".
`RecordResumer`, which records the label for the `--until-stop` waiter, leaves "running and paused runs ... unmarked".
So the same resume from paused writes `by` on a plain step but not on an `--until-stop` step.
The decision record limits the label to "resumes of a halted run" and "A step on a halted run records its label on the status", and a paused run is not halted.
A run paused after a goto already carries the goto's `by` on the goto entry, which is the entry card 10's settler reads, so nothing needs the paused case.

### [MEDIUM] New batten test duplicates TestBattenDriverDeps' surface

**Location:** `_lyx/plan/06-caller-label-wiring.md`, Create `plan:internal/battencli#TestBattenAutomaticMoves_LabelShed`.

**Issue:** `internal/battencli/driverdeps_test.go` already holds `TestBattenDriverDeps`, "the seams batten fills the shed driver with", which calls `c.driverDepsFor()` on `newFakeReceiver` and asserts each seam's wiring.
`newFakeReceiver` leaves `untilStop` nil and gives the receiver temp status paths, so `deps.Step` reaches `RunUntilStop` there.
The label `shed` that batten's `Goto` and `Step` seams now pass is one more property of that same wiring.
`PATTERN-test-economy` makes it an assertion in the existing test, not a new top-level test that also raises `internal/battencli`'s budget row to 84.
The loom-side `TestGoDriverSeams_LabelShed` is not affected: no existing `internal/loomcli` test calls `goDriverDepsFor`.

## Rejected

- Card 6's hidden `goto --by` flag against the audience bullet of `pattern/PATTERN-cli-cobra.md`: that bullet governs commands, not flags. The repo already hides flags the same way (`step`'s `LoopDetachedFlag`, `reed remove --wait-pid`, `loom start --no-attach`). The Intent states the bound: only `shed` is admitted, the label enters no Go decision, and the flag can only mislabel a report row. Not a defect.
- Card order: card 8 uses card 7's `Sources.Branches` and `SnapshotAt`. Card 10 uses card 5's `By`, card 3's `shedrun` settle format and card 9's `battenRowHalts`. Card 11 uses card 1's `durfmt`. Card 12 uses card 2's `sleepjournal` and card 11's `RenderTimingMarkdown`. Card 6 uses card 4's `CallerLabel` and card 5's `CallerShed`. No card depends on a later one.
- Reachability: the `--by` refusal row `--to B --by orch` in `TestGotoCmd_RefusesMissingOrUnknownTarget` reaches it, since `B` is a producer in `gotoSpec` and the missing-`--to` check passes. `TestRecordResumer` reaches the undecodable-status error by calling `RecordResumer` directly. Card 12's flag rows reach `parseTimingOptions` in a temp directory before any cwd resolution, as convergence's rows do today.
- Card 8 counting batten's `Run-Shed` entries under the child's phase rather than `other`: the decision record's table uses `Run-Shed` as the boundary between `setup` and `land`, and the card states the rule. Counting them under `other` would put a finding on every run. This fills a gap; it contradicts nothing.
- Card 10 reading batten's settle file at the commit holding the run's batten record rather than at the tag: batten's run directory is committed on the records branch the prime writes, not on the pair branch the tag points into. Card 10's choice is where that file exists.
- Card 3 listing whole files (`internal/sheddriver/drive.go`, `internal/arbiterstop/holdreading_test.go` and others) beside member glyphs: the rubric's member-glyph rule fires only for an `Edit` card whose targets are only file paths. Card 3 also lists members, and the gate passes it.
- Verify coverage: one module (`tools/` included under `./...`), with the tiers in the template's order.
- Test budgets: the rows each card raises match the current counts (`reportengine` 8, `reportcli` 3, `shedrun` 24, `agentname` 8, `statuspane` 6, `shedengine` 140 of 141).
- Fidelity of cards 7 to 11: every decision is carried by a card. That covers the phase table, the waits and their order, sleep subtracted first, attempts and the absent-time finding, the settler order, the record join, seconds, output keys, and the markdown tables with sleep columns. Every constraint is carried too: the allowlist gains only `durfmt`, the leaf entries, the spawn logging, the sandbox lines and the docs.
