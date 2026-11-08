```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over `buttonContext`, `pairIsBeam` and the new test rows.

**Per function and branch**

- `buttonContext`, `prime=true`: `pairIsBeam` is skipped and the place is prime. Correct.
- `buttonContext`, `pairIsBeam` returns an error: the error is returned unwrapped, since `pairIsBeam` already wrapped it, and reed renders no buttons. Correct as designed, but see F1.
- `pairIsBeam`, origin read fails: wrapped and returned. Correct.
- `pairIsBeam`, origin found with `Beam` true: returns true without touching the board. Correct.
- `pairIsBeam`, origin absent, and the board file or its directory absent: `Store.Load` sets an empty list and returns nil when `board.json` and the legacy files are missing (`internal/boardengine/store.go:181-256`). The loop finds nothing and returns false, so the pair reads as task. Correct. I did not open `readLegacyFile`, so I assumed it reports "absent" for a missing legacy file or missing directory.
- `pairIsBeam`, board holds the slug as a project with `Beam` true, whether the status is open or done: returns true. Correct for the reported case. A done beam entry still reads as beam, which matches the origin-mark behaviour.
- `pairIsBeam`, board holds the slug as a non-project or non-beam entry: returns false without scanning further. Correct, because slugs are unique on the board.
- `pairIsBeam`, empty board and empty `WorktreeName`: no match, false. Correct.
- `pairIsBeam`, board unparseable, version mismatch or lock failure: returns an error. See F1.

**Findings**

- F1, LOW, design, `internal/hubgeom/buttons.go:67-70` (evidence `internal/reedengine/buttonrow.go:75-79`):
  - **Defect:** a board read failure now turns every non-beam task session's button render off. Before the diff, a task pair whose origin record was found without the beam mark never touched the board and kept its task buttons. Now a corrupt `board.json` makes `Buttons()` fail, and `composeButtonRowLocked` renders no buttons and pins an empty spec.
  - **Why LOW:** the board is only a fallback for the beam flag. The failure also surfaces only on a hand-corrupted board, and it is logged at Warn.
  - **Fix:** log the board read error at Warn and return false (task), or return the error only when no origin record was found.
  - **Test:** the `UnparseableBoardIsAnError` row at `hubgeom_test.go` would then change.

I found no other defects in my lens.

**Tool calls:** none. The manifest already held every definition I needed.
