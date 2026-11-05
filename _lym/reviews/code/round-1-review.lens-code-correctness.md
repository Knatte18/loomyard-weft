```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, with a trace per function or branch, then the findings.

## Trace list (function or branch, case run, verdict)

- **`CheckSeal`**
  - Case: `--discard` amendment with an edited file. Returns Empty with no check. Correct.
  - Case: `Unsealed` set. Returns Empty. Correct.
  - Case: row not past Drift-Expect. Returns NotYet. Correct.
  - Case: no seal commit and no file, then no seal commit and a file. Returns Empty, then a `--discard` Fault. Correct.
  - Case: seal commit whose `CodeHead` differs from the baseline head. Returns a `--discard` Fault. Correct.
  - Case: current file edited. Returns a `--restore` Fault with `Sealed` still the seal's copy. Correct.
- **`ReadRecords` / `WriteRecords` / `RestoreRecords`**
  - Case: no Drift-Records commit. Reads the files as they are, Committed false. Correct.
  - Case: mismatch on write. Refuses, naming `--restore`. Correct.
  - Case: restore of a file the commit lacks. Removes it. Correct.
  - Case: commit failure after the writes. Leaves a mismatch, which D6 documents. Correct.
  - Case: nothing staged. No commit, and the digest is unchanged. Correct.
- **`DiffCaptures` / `listLines` / `diffLines`**
  - Case: CRLF input, trailing newline, empty file. All correct.
  - Case: repeated key. Gets `N`. Correct.
  - Case: keyed insert. Moves no unkeyed identity. Correct.
  - Case: added or removed case. Correct.
  - Case: a pattern whose `key` group matches empty (F2). Wrong.
- **`ParseExpectation` / `Cover` / `Merge` / `Uncovering`**
  - Case: `n` absent, `n` above the baseline count, `n` below 1. Correct.
  - Case: open range `12-`. Covers unkeyed cells only. Correct.
  - Case: a line entry against a keyed cell. Not covered, correct.
  - Case: command-mode `keys` entry. Refused, correct.
  - Case: empty document and EOF. Parse as the empty expectation, correct.
- **`Check`**
  - Case: scope off, deferred, waived. Skip, correct.
  - Case: never counted, baseline missing, digest changed. Baseline fault, correct.
  - Case: records mismatch. Check goes on over the committed copies, correct.
  - Case: capture failed. Capture fault, correct.
  - Case: command mode. Runs the compare and takes no capture, correct.
- **`NewDriftGate`**
  - Case: unexpected cell with a seal or records fault (F1). Wrong.
  - Case: capture fault with no other fault. Fails, correct.
  - Case: command mode. Passes, correct.
- **`DriftVerify.pass`**
  - Case: the Done short-circuits in D4 order. Correct.
  - Case: mismatch before the scope-off Done. Correct, because the committed copy is read.
  - Case: `AwaitedAt` empty. Stuck, correct.
  - Case: unchanged head after a passing check. Done, and the baseline digest is checked first, correct.
  - Case: judge mark open and snapshot gone. Records `SnapshotLost` and gets a Stuck, then takes a new snapshot on resume. Correct.
  - Case: `recordUnsealed` with a file present. Left unmarked so the seal faults, correct.
- **`checkAndJudge` / `storeRulings`**
  - Case: unexpected cell. Stuck naming amend and the fix route, correct.
  - Case: stored `matches`, and stored `does not match`. Reuses, and Stuck with no judge. Correct.
  - Case: all rulings `matches`. Done, and `judgement.yaml` removed. Correct.
  - Case: a `does not match` ruling. Stores the tuples, closes the mark, removes `judgement.yaml`, then Stuck. Correct, and it matches the third Added-after-Discussion entry.
  - Case: an open-range note that the request did not list. Dropped, correct.
- **`passCommand` / `storeCaseRulings` / `storedCaseRulings`**
  - Case: compare exit 0. Records the pass and closes the mark, correct.
  - Case: stored not-matching ruling at the same head with no later amendment. Stuck with no judge, correct.
  - Case: ruling and amendment in the same second. Counts as amended, which the doc states. Correct.
- **`judgeSessionChanged` / `Changes` / `checkoutChanges` / `scratchChanges`**
  - Case: judge edits a tracked file, deletes an untracked file, adds a file, or reverts a change. Each correct.
  - Case: scratch directory gone. Left out, correct.
  - Case: moved records HEAD. Checked through `RecordsChangedSince`, with the exempt loom paths removed. Correct.
  - Case: `judge-request.json` rewritten on resume. It sits under the ephemeral `.lym` directory (`paths.go:66`), outside the guard, so there is no false Stuck. Correct.
- **`Amend`**
  - Case: none or several forms. Refused, correct.
  - Case: row not blocked. Refused, correct.
  - Case: `--restore` with nothing to restore. Refused, correct.
  - Case: `--restore` after a records mismatch plus a seal fault. Restores both in order, correct.
  - Case: `--entries`. Drops the covered rulings, correct.
  - Case: `--discard`. Correct.
  - Case: a records mismatch at Drift-Expect (F3). Wrong.
- **`Defer` / `deferAdmitted`**
  - Case: prefix halts at the listed rows. Admitted, correct.
  - Case: baseline fault with no prefix. Admitted, correct.
  - Case: scope off or already deferred. Refused, correct.
- **`DriftExpect.pass`**
  - Case: sealed, scope off, deferred, never counted, baseline fault, moved head. Each correct.
  - Case: `Diff` empty. Recorded first, correct.
  - Case: closed mark with a valid file and no session ever opened (F4). Wrong.
  - Case: interrupted session. Resumed through the agent producer, correct.
  - Case: exhausted gate. The mark is closed and the stop chain runs, correct.
- **`DriftBaseline.recordScope`**
  - Case: config scope off. Correct.
  - Case: config scope partial. Recorded full with one Warn, correct.
  - Case: scope already recorded. Kept, correct.
- **`strandRoleStopper`**
  - Case: exact-role match, retiring strand, non-live strand. Correct.
- **`trailercommit.go`**
  - Case: invalid trailer key or value. Refused before anything is staged, correct.
  - Case: no commits in the warp. No `Warp-SHA` line, correct.
  - Case: `FindTrailerCommit` oldest and newest. Correct, assuming `Log` returns newest first.
  - Case: `AnchoredFileAt` for an absent path. `found` false, correct.

## Findings

**F1 — MEDIUM, design.** `internal/driftengine/check.go:370-377`.
- `NewDriftGate` fails only when `clearable` is true, meaning no skip and no baseline, records or seal fault.
- With a records or seal fault present, unexpected cells and a failed current capture are dropped. The gate passes, and the only trace is a `logger.Info` line.
- D8 says the gate reports a seal or records failure without failing and still checks containment against the committed copies.
- So the writer never sees its unexpected cells, and never sees the seal fault, which no session could clear anyway. D8's `lym drift check` path through `Findings()` does show both.
- The fix is to fail on unexpected cells and a capture fault whenever the check ran. When a records or seal fault is present, add it to the findings text instead of withholding the failure.

**F2 — LOW, design.** `internal/driftengine/diff.go:146-148`.
- A key pattern whose `key` group matches the empty string produces `Identity{Key:"", N:n, Line:0}`.
- Everything downstream treats an empty `Key` as unkeyed (`Cover` at `expectation.go:324`, `coversLine` requiring `Line >= 1`), so such a cell can never be covered.
- Several empty-key lines also collide when they share `N`.
- The fix is to treat an empty `key` match as no match in `listLines`, or to refuse it in `CompileKeyPatterns`.

**F3 — LOW, design.** `internal/driftengine/recordsguard.go:149` and `internal/driftengine/amend.go:32`.
- `WriteRecords` refuses on a mismatch naming `lym drift amend <slug> --restore`.
- Drift-Expect and Drift-Baseline write records too, and `amendRows` excludes both, so `Amend` refuses there.
- At those rows the mismatch is a plain error, not a Stuck handed to the parent, and its named way forward is itself refused.
- The fix is to make a records mismatch at the Drift-Expect and Drift-Baseline writes a parent halt, or to add `expectRow` to `amendRows` for `--restore`.

**F4 — LOW, design.** `internal/driftengine/expect.go:129-137` and `:233`.
- D6 allows sealing an existing `expected.yaml` without spawning only when the expect session ended normally.
- `runAgent` resets the mark to the zero `SessionMark{}` on close, so "closed after a normal end" cannot be told from "never spawned".
- A valid `expected.yaml` that exists before the agent ever ran, such as a hand-written or inherited file, is sealed with no agent.
- The fix is to keep a closed mark that records the session ended, for example `Open:false` with `OpenedAt` kept, and require it before sealing an unspawned file.

## Tool calls

1. `grep` in `internal/driftengine/paths.go` and `judge.go`, to learn where `JudgeRequest` lives and whether the snapshot guard covers it. My context lacked `paths.go`.
