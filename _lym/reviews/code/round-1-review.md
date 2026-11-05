---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/driftengine/check.go:370
    origin: lens:code-correctness
    summary: The drift gate passes on unexpected cells and a failed current capture whenever a records or seal fault is also present, so the writer never sees them
    fix: In NewDriftGate, narrow clearable to report.Skip == "" && report.BaselineFault == "" so a records or seal fault no longer turns off the failure. Keep failing with the report's findings on a capture fault or on any unexpected cell, and keep the records and seal fault text in those findings. Add rows to the table in internal/driftengine/check_test.go for a seal fault and a records mismatch that also carry an unexpected cell, each with gatePasses false.
  - id: F2
    severity: LOW
    class: consistency
    location: commits e2dcd747 and b7f922f6 (Pattern-Departure trailers)
    origin: lens:code-affected-set
    summary: Two commits carry a Pattern-Departure trailer naming PATTERN-fabric-vocabulary, which is not an entry in PATTERN.md or pattern/
    fix: The commit messages cannot be rewritten inside the range. In the fixer report, record that the departures (CodeHead, RecordsChangedSince, LoomRecordsDirs and RecordsPathsChangedSince in place of the card-named WarpSHA and Weft spellings) rest on PATTERN-fabric-boundary, whose rule the side-named identifiers would break, and make no code change.
  - id: F3
    severity: LOW
    class: design
    location: internal/driftengine/diff.go:146
    origin: lens:code-correctness
    summary: A key pattern whose key group matches the empty string yields an Identity with an empty Key and Line 0, which no expectation entry can cover and which collides with other such lines
    fix: In CompileKeyPatterns nothing can tell the match is empty, so in listLines treat a match whose key group is empty as no match and fall through to the next pattern or to the unkeyed branch. Add a row to the table in internal/driftengine/diff_test.go for a pattern that can match an empty key, expecting an unkeyed identity.
  - id: F4
    severity: LOW
    class: design
    location: internal/driftengine/recordsguard.go:149
    origin: lens:code-correctness
    summary: A records mismatch at Drift-Expect or Drift-Baseline is a plain error naming lym drift amend --restore, a verb that amend refuses at those rows
    fix: Make Drift-Expect and Drift-Baseline turn a records mismatch into a parent halt instead of an error, and add expectRow to amendRows in internal/driftengine/amend.go so that --restore is admitted at a blocked Drift-Expect. Add a row to TestAmend for --restore at Drift-Expect and one to TestDriftExpect for the mismatch Stuck.
  - id: F5
    severity: LOW
    class: design
    location: internal/driftengine/expect.go:129
    origin: lens:code-correctness
    summary: Drift-Expect seals a valid expected.yaml with no spawn whenever the session mark is closed, which a run that never spawned the agent also shows, so D6's normally-ended-session condition is not enforced
    fix: Keep a closed mark that records the session ended, such as Open false with OpenedAt kept, when runAgent closes it at expect.go:233. In pass, seal an existing file without spawning only when that closed mark is present, and spawn the agent otherwise. Add a row to TestDriftExpect for a valid file present with no session ever opened, expecting a spawn.
  - id: F6
    severity: NIT
    class: consistency
    location: internal/driftengine/verbs_test.go:56
    origin: lens:code-tests
    summary: TestDeferWaive keeps the name of the waive verb that D12 removed
    fix: Rename TestDeferWaive to TestDefer in internal/driftengine/verbs_test.go.
---

### [MEDIUM] The drift gate withholds its failure when a records or seal fault is present

**Location:** `internal/driftengine/check.go:370`
**Issue:** `NewDriftGate` sets `clearable` only when the report holds no skip and no baseline, records or seal fault, and fails only when `clearable` is true. With a records or seal fault, an unexpected cell or a failed current capture makes the gate pass, with one `logger.Info` line as the only trace. D8 says the gate reports a records or seal failure without failing and still checks containment against the committed copies, so the writer must still be told about its unexpected cells. No row of the `TestCheck` table combines a fault with an unexpected cell, so nothing catches it.

### [LOW] Pattern-Departure trailers name an entry that does not exist

**Location:** commits `e2dcd747` and `b7f922f6`
**Issue:** Both commits cite `PATTERN-fabric-vocabulary`, and `PATTERN.md` and `pattern/` carry no such entry. The rubric treats a trailer naming an unknown entry as a finding. The renames themselves are sound, since `pattern/PATTERN-fabric-boundary.md:24` bars a side-named identifier outside the owner set.

### [LOW] An empty key group makes a cell no expectation can cover

**Location:** `internal/driftengine/diff.go:146`
**Issue:** A pattern that can match an empty `key` group gives a line `Identity{Key: "", N: n}`. `Cover` and `coversLine` treat an empty `Key` as unkeyed and require `Line` of at least 1, so the cell is always unexpected, and several such lines share an identity when their `N` agrees. Key patterns come from config, a trust boundary, so the case is worth a guard.

### [LOW] A records mismatch at Drift-Expect or Drift-Baseline names a verb amend refuses

**Location:** `internal/driftengine/recordsguard.go:149`
**Issue:** `WriteRecords` refuses on a mismatch with a message naming `lym drift amend <slug> --restore`. Drift-Expect and Drift-Baseline write records, and `amendRows` in `internal/driftengine/amend.go:32` leaves both out, so `Amend` refuses there. The refusal also reaches the run as an error instead of a Stuck for the parent. The way forward the message names is itself refused, which breaks the way-forward rule.

### [LOW] Drift-Expect's normal-end condition for sealing without a spawn is not enforced

**Location:** `internal/driftengine/expect.go:129`
**Issue:** D6 allows sealing an existing `expected.yaml` without a spawn only when the expect session ended normally, as its mark records. `runAgent` closes the mark to the zero `SessionMark{}`, so a closed mark after a normal end looks the same as a mark never opened. A valid `expected.yaml` that exists before any agent ran is sealed with no agent session.

### [NIT] A test keeps the name of a removed verb

**Location:** `internal/driftengine/verbs_test.go:56`
**Issue:** `TestDeferWaive` tests only `Defer`, since `lym drift waive` was removed by D12.

## Rejected

- The code-tests lens's unverified point that the D15 reinterpretation flags might lack a row each: `internal/driftengine/diffmode_test.go:86` to `:102` holds a row for a `compare` with no `diff`, one for `{skip_file}` in both steps and one for `scope: partial`.
