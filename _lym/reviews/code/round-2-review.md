---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/driftengine/capture.go:216
    summary: backUpBaseline leaves the pair half-moved when the metadata rename fails, and the next capture then deletes that orphaned metadata
    fix: In backUpBaseline, when the os.Rename of the metadata directory fails with an error other than os.ErrNotExist, rename the backup back to baselineDir before returning the error, and join the rollback's own error to it with errors.Join. That leaves baseline and baseline.meta together, so the next capture's empty-baseline cleanup never removes the metadata of a baseline that was moved. No test row is needed, since a failing rename of one sibling cannot be induced portably in a tier 1 test.
---

### [LOW] backUpBaseline can leave the baseline moved and its metadata behind

**Location:** `internal/driftengine/capture.go:216`

**Issue:** `backUpBaseline` renames `baseline` to `baseline.N` first, then renames `baseline.meta` to `baseline.N.meta`.
When the second rename fails for any reason but a missing source, the function returns the error with `baseline` already moved and `baseline.meta` still in place.
`RunCapture` records the capture as failed and returns.
`baseline` no longer exists at that point, because `MkdirAll` at the end of `backUpBaseline` is skipped by the early return.
The next `RunCapture` finds an absent baseline directory and takes the `len(entries) == 0` branch, which runs `os.RemoveAll(baseline.meta)`.
That deletes the metadata that belongs to `baseline.N`, whose pair is now permanently split.
The same-number mismatch cannot arise, because `nextFreeBackup` rejects a number whose `.meta` already exists, so the exposure is limited to this failure path.
Metadata is kept for the operator and the capture record already names the commit, so the loss is small, but the "pair is never half-moved" intent of the new backup code does not hold on this path.

## Rejected

- Focus 1: `unchangeableFindings` in `internal/driftengine/check.go` leads with `r.faultLines()`, so a records fault and a seal fault found beside the unchangeable cells are named, as the `NewDriftGate` doc comment and `doc.go` promise.
  The baseline and capture faults cannot be set on that path, because the gate returns earlier on either.
  `TestDriftGateUnchangeableCells` has a row for the records fault and one for the seal fault, each asserting `--restore`, the cell key and `metadata:` and a Terminal result.
  Not a defect.
- Focus 2: a leftover `baseline.N.meta` with no `baseline.N` makes `nextFreeBackup` skip that number, so no meta directory is overwritten or merged; the capture test for a taken meta dir pins it.
  A stale `baseline.meta` beside a missing or empty baseline is removed before the capture, which a capture test pins.
  A `baseline.meta` beside a non-empty baseline moves with it, so that pair stays matched.
  The only half-moved case is the rename failure in F1.
