---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/reportengine/timing.go:341
    summary: pairSlug re-implements fabricengine.WeftWarpSlug, which card 8 names for the pair-branch parse and which is documented as the inverse of RecordsBranchName
    fix: In internal/reportengine/timing.go delete pairSlug and call fabricengine.WeftWarpSlug(branch.Name) in selectInFlight (it returns the same slug and false for an empty one); drop the strings import if nothing else uses it, and reword the selectInFlight doc comment and the Timing paragraph in internal/reportengine/doc.go that say the name is the one RecordsBranchName forms so they say WeftWarpSlug parses it.
  - id: F2
    severity: LOW
    class: design
    location: internal/reportengine/timing.go:399
    summary: bucketRuns carries a loop whose only effect is phaseTime[phase] += 0, an unexplained write that makes a held phase appear as 0 in the bucket
    fix: In bucketRuns replace the loop over run.Phases with an explicit presence write guarded by a one-line comment saying a phase the run holds but spends no time on in this bucket stays present as 0, for example `if seconds != nil { if _, seen := phaseTime[phase]; !seen { phaseTime[phase] = 0 } }`.
  - id: F3
    severity: LOW
    class: design
    location: internal/reportengine/timingspans.go:252
    summary: Sleep, snapshot and attempt times keep sub-second parts while history stamps are whole seconds, and every figure is truncated on its own, so phases plus sleep, or active plus waits plus sleep, can miss the span by a second or more on real data; read from the code, not reproduced
    fix: In timeRun truncate opts.Sleeps and the snapshot to whole seconds before they enter the span arithmetic (Truncate(time.Second) on each endpoint, after which unionIntervals and clipIntervals are unchanged), and likewise truncate the endpoints runWaits and runAttempts take from change.At, the parent-review request and verdict times and spawned_at, so every interval boundary is a whole second and the per-figure truncation sums exactly; add one TestComputeTiming row with a sleep carrying a fractional second asserting phases plus sleep equal span_seconds.
---

### [LOW] pairSlug duplicates fabricengine.WeftWarpSlug

**Location:** internal/reportengine/timing.go:341
**Issue:** Card 8 selects pair records branches by "a branch whose name `fabricengine.WeftWarpSlug` parses, the slug being what it returns".
`fabricengine` already declares that parse, documented as the inverse of `RecordsBranchName`.
The diff adds a second one, `pairSlug`, that cuts `RecordsBranchName("")` off the branch name.
The two agree today; a change to the suffix rule in `weftwiring.go` would have to be repeated here.

### [LOW] No-op write in bucketRuns

**Location:** internal/reportengine/timing.go:399
**Issue:** The loop `for phase, seconds := range run.Phases { if seconds != nil { phaseTime[phase] += 0 } }` adds nothing; its only effect is to create the map key so a held phase renders as 0 rather than being absent.
That is the plan's intent (a phase a run in the bucket holds is present), but nothing says so and the form reads as dead code.

### [LOW] Whole-second sums can miss the span

**Location:** internal/reportengine/timingspans.go:252
**Issue:** Card 8 and 9 require the phases plus sleep, and active plus waits plus sleep, to equal the span.
Row stamps are RFC 3339 whole seconds, but `opts.Sleeps` come from journal microseconds, `SnapshotAt` carries nanoseconds and attempt and round times come from records that may too.
`timeRun` truncates each phase, each wait kind, the sleep and the union of waits separately, so the truncations do not add back to the truncated span when any boundary has a fractional second.
The fixture rows use whole seconds, so the tests do not reach it.

## Per-card checks

- Cards 1 to 3: the old symbols are gone (no `statuspane.Format*`, `parseSleepJournal`, `journalEntry`, `sleepOperation` or `sheddriver.Settle*` constants outside their new homes), and the callers retarget to `durfmt` and `shedrun`.
  The cards are Create/Delete pairs, not Rename pairs, so the `git mv` mechanic does not apply to them.
  The test-budget rows match the new top-level test counts.
- Cards 4 to 6: `CallerLabel` is the sole classification and sits in `agentname`; `shedengine` and `shedverbs` take the label as a string.
  `By` and `PendingBy` are `omitempty`, so older records decode and re-encode unchanged.
  `--by` is hidden, refuses anything but `shed` before the status file is read, and has its row in `refusal-spec.md`.
  Both driver seams and `childGotoArgv` pass `shed`.
- Cards 8 to 10: sleep is cut out of every interval and every wait before the waits are netted, waits are netted by kind in the listed order, medians take the mean of the two middle values rounded down and ignore runs in flight, and open spans run to the snapshot.
  Apart from F3 no double counting was found.
- Cards 11 and 12: flags, run selection and output match the plan, and `doc.go`, `docs/overview.md`, `SANDBOX-CORE-SUITE.md` and the budget file are updated.
- Test economy: the new tests are the ones the cards name, each pinning a public behavior once; no duplicate or per-helper test found.
- The tests of the touched packages pass under `lyx gate test`.
