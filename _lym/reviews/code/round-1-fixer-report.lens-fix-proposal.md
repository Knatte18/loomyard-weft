```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals that the fix carried out as proposed (F1, F3–F7; F2 was changed, so it is out of this lens).

**1. MEDIUM: F1, leaving an absent beam out of `Beams`**
- **Changed text:** the `readBeamMode` early error in `internal/hubgeom/beammodes.go`, and the new sentence in `internal/hubgeom/doc.go`, "so the board files never show its parent's values as its own runs' values".
- **Summary:** a left-out beam can still read as taking the hub-wide default.
- **Case:** machine B renders. Beam X's worktree is only on machine A, where it sets `autonomy: true`. The root and every beam present on B take the hub-wide `off`.
- **What happens:** `everyPlaceHubWide` looks only at `Root` and the beams in `Beams`, so it drops Autonomy from every mode line.
- **What the reader sees:** the header gives "Hub-wide default: autonomy `off`", and the legend says "A setting that the root and every beam take from the hub-wide default is stated here alone". X's section has no mode line, so X reads as `off`, though its runs read `on`.
- **Evidence:** `modeHeader` and `everyPlaceHubWide` in `internal/boardengine/beammodes.go`. Nothing in the README says that a beam with no line is unknown.
- **Bound on this:** the old code reached the same omission in this case, so the fix did not introduce it. It leaves part of F1's wrong-value premise unresolved.

**2. LOW: F1, reporting the absent worktree through the error path**
- **Changed text:** the `fmt.Errorf("... is not on this machine")` return, which `BeamModes` turns into `logger.Warn`.
- **Summary:** an expected condition now warns on every board render.
- **Case:** on a hub that spans machines, each board write prints one warning per open beam whose worktree is elsewhere. The message is "the beam's modes did not read", as if something had failed.
- **Evidence:** the `BeamModes` loop in `internal/hubgeom/beammodes.go`. In `internal/logger/logger.go`, the default threshold is warn, so `Warn` reaches stderr. Before the fix, this case returned without a warning.

F3, F4, F5, F6 and F7: no findings.

Tool calls:
1. Read `ReadOriginFor` in `internal/fabricengine/origin.go`, to see whether leaving the beam out also loses a parent branch that is readable without its worktree. My context did not hold it. No finding came of it.
2. Grepped `internal/logger` for `Warn`, and `pattern/` and `cmd/lym/main.go` for the beam-mode wording. My context did not show where `Warn` goes or what the pattern entry says.
