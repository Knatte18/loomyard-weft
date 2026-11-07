```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens review of the committed diff.

**Per function and branch: the case I ran, and whether it is correct**

- `metaDirFor`, `NewPaths`: `baseline` gives `baseline.meta`, and a current capture `X` gives `X.meta`. Correct.
- `renderCommand`: a command holding both placeholders gets both replaced, and a command holding neither is unchanged. Correct.
- `runStep` `MkdirAll(MetaDir)`: `RunCapture`, `CaptureCurrent` and `RunCompare` all set `MetaDir`, so an empty path is not reached. Correct.
- `backUpBaseline`, no metadata directory: the `ErrNotExist` is ignored and the backup proceeds. Correct.
- `backUpBaseline`, backup slot taken: see finding 2. Not correct.
- `backUpBaseline`, empty baseline directory: the old metadata directory stays in place and the next capture writes into it. See finding 2.
- `checkMetadata`:
  - Blank file or key: refused.
  - Bad glob: `path.Match(pattern, "")` still reports `ErrBadPattern`.
  - Glob matching no baseline case: refused.
  - `keys` under `diff: command`: refused.
  - Correct.
- `Volatile.union` with empty or nil inputs: returns nil slices without panicking. Correct.
- `Check`: `report.Metadata` is the union of the scope record's and the expectation's marks, and `classifyCurrent` drops their cells. A skip or fault that returns early leaves `Metadata` empty, which is acceptable. Correct.
- `seal`:
  - With marks, they are written to the scope record before the seal commit.
  - With no marks, the scope record's existing marks are kept.
  - A parse refusal is discarded as `_`, but `seal` is only reached after validation.
  - Correct.
- `settleUnchangeable`, rule 1: with `baseHead == ""` or equal to `head`, nothing is marked. A seven-hex-digit decimal such as `0123456` could match, but both the old and new value must match, so the odds are negligible. Correct.
- `settleUnchangeable`, rule 2:
  - A second capture that fails marks nothing.
  - A cell that is absent from the second capture, or has a different `Kind` or `New` value, is marked.
  - Correct.
- `NewDriftGate`, mixed failure: the writer's findings omit the unchangeable cells, and the gate re-prompts. Correct.
- `NewDriftGate`, only unchangeable cells: returns a `Terminal` failure. `wait.go:1881` records it and `wait.go:589` finalizes at once. The gate side is correct.
- `unchangeableFindings`: drops the records and seal faults. See finding 1.
- `Verdict`: `driftRecordsDeps` wires `TrailerCommit` (`driftwiring.go:68`), and `baseline.CodeHead` is the commit's Warp-SHA. A missing commit gives an empty `baseHead`, so the line is skipped. Correct.
- `renderVerdict`: the `Baseline:` line is added only on the counted-scope branch, and `metadataLines` returns nil when there are no marks. Correct.
- `storedCaseRulings` and `storeCaseRulings`: a ruling on a marked case counts as `matches` and is not stored. Correct.
- `Amend`: metadata-only `--entries` parse through `ParseExpectation` and are merged in `Check`. `dropCoveredRulings` drops no rulings for them, since `Cases` is empty. A stored ruling for a marked case under `diff: command` is also ignored by `storedCaseRulings`. Correct.
- `checkResult` in `driftcli`: uses `Unexpected()`, so it hides unchangeable cells. These are only set by the gate, so `lym drift check` shows no difference. Correct.

**Findings**

1. **MEDIUM, design.** `check.go` `unchangeableFindings`, around lines 482–490.
   - The `Terminal` findings omit `RecordsFault` and `SealFault`; only the cells and the way forward are rendered.
   - The `NewDriftGate` doc comment, and `doc.go`'s Drift gate text, say the failure carries findings that also name a records or seal fault found beside them. That holds on the non-terminal path via `Findings()`, not on the terminal one.
   - A records mismatch or a failed seal check that coincides with metadata-only cells is therefore hidden from the parent's one question.
   - The fix is to render the faults, or `report.Findings()` minus the unexpected-cell block, into the terminal findings.

2. **LOW, design.** `capture.go:208-216`, `backUpBaseline` and `nextFree`.
   - `nextFree` checks only `baseline.N`, never `baseline.N.meta`.
   - A leftover `baseline.N.meta` directory with content (for example from a prior failed move) makes the metadata `os.Rename` fail after the baseline has already been moved. The capture then errors with the baseline moved and its metadata left behind, a half-moved state.
   - Also, when the baseline directory is empty, an old `baseline.meta` is not cleared, so stale provenance can sit beside a new capture. The metadata directory is never emptied before a capture. Neither case affects what drift compares.

3. **LOW, design.** `check.go` `settleUnchangeable`, second-capture rule.
   - Any unexpected cell whose value differs in a second capture is marked unchangeable and sent to the parent.
   - A genuinely nondeterministic or flaky result the writer's change caused is handled the same way as provenance. That fits the entry's "cells that differ between two captures of the same code", but the findings text calls it "capture metadata", which it may not be.
   - This is a wording and design note only, not a defect.

**Tool calls**
1. `grep` for `gateTerminal` in `shuttleengine` — confirm how a terminal gate result finalizes. My context only had the `wait.go` lines.
2. `grep` for `Verdict(` callers — find the deps `Verdict` is called with.
3. `grep` for `runStep(` and `stepRun{` — confirm every caller sets `MetaDir`.
4. `sed` of `capture.go` 196–240 — read `backUpBaseline` and `nextFree` in full.
5. `grep` for `TrailerCommit` outside `driftengine` and `fabricengine` — check the seam wiring.
6. `grep` for `driftRecordsDeps` — confirm `TrailerCommit` is wired for `Verdict`.
