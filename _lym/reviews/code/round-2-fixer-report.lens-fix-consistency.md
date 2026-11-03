```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (HoldsReexec hold now lasts until the sequence ends) for sibling and twin disagreements.

**LOW** — `internal/beamcli/doc.go:147`: the prose still describes the hold as "skips it while `Carrier.HoldsReexec` reports a sequence past its trigger wait".
- The new hold also covers a terminal request while the sequence lifts its entry and sends its notice.
- That is still "in flight past its trigger wait", so the sentence is not false.
- It is incomplete next to the updated `hibernateengine/doc.go:57` and the `HoldsReexec` doc comment, so a reader of the beamcli doc would not learn that the hold outlasts the terminal write.
- Optional: add "until it has sent its notice".

Checked and consistent:
- `carrier.go` doc comment, `hibernateengine/doc.go:57` and the `HoldsReexec` body agree.
- `hibernateengine/doc.go:58` ("Bound: a re-exec that passed the check in the instant the sequence moved past waiting…") is a different race, the waiting→pausing instant. The change leaves it correct.
- `holdReexec` in `beamcli/dobbydeps.go:219` is generic, with no terminal-phase assumption. Its only call site, line 199, passes `carrier.HoldsReexec` unchanged.
- `carrier.go` has no remaining use of `Terminal()` in `HoldsReexec`, and the removed import-free expression leaves no dead code.
- The test at `carrier_test.go:184` matches the new semantics, and the `waitTerminal` comment is not contradicted.

Tool calls:
1. `grep` for `HoldsReexec` and the phrase "past its trigger wait" across the repo, because my context did not show other references outside the engine package.
2. `grep` for `func holdReexec` in `internal/beamcli`, because my context lacked the wrapper that consumes `HoldsReexec`.
