```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens on the diff (`checkLivenessTick`'s `markedStopped` branch, `markedStopped`, and the Wait and doc claims).

**Per-branch list**

1. `checkLivenessTick`, untracked strand, files present. I took Status without the strand and all outputs on disk. The result is `OutcomeDone`, because the file check comes before the new branch. Correct (`wait.go:1267`).
2. `checkLivenessTick`, untracked strand, files missing, handle marked. I took the `stopMarked` handle with an empty Status. The result is `OutcomeDied`, `nil`. `finalizeAndRemoveDiedWindow` then reaches `finalize`, which records `stopOutcome(died, files)`, which is `stopped` (`wait.go:1796`). Correct. I did not read `finalizeAndRemoveDiedWindow`, so any window removal it does after a stop is unverified.
3. `checkLivenessTick`, untracked strand, files missing, handle unmarked. This still returns `errStrandNotTracked`. Correct.
4. `markedStopped` against `recordMu`. I listed every `recordMu` site. `Stop` releases the lock before `removeStrandIfPresent`. `finalize` (`wait.go:1794-1801`), `markStarted` (`wait.go:1371`), `dialog.go:131`, `hold.go:165` and `hold.go:189` take it only briefly. `checkLivenessTick` is not called under any of them, and none nests with `markedStopped`. There is no deadlock and no race. Correct.
5. `Stop` with an already-terminal record. `stopMarked` is not set, because the mark is inside the `!isTerminalOutcome` block (`stop.go:38-40`). The strand is still removed, and a Wait on that handle would still hit `errStrandNotTracked`. This is not reachable in practice: a terminal record means Wait already finalized. Correct.
6. `Stop` with the files all present. The mark is set and the outcome stays `running`. The liveness check returns `Done` first. Correct.
7. `awaitStartup`'s use of `checkLivenessTick`. No handle exists yet, so no `Stop` can have marked it. The new branch is unreachable there. Correct.
8. The new test. The first Status is consumed by `Stop`'s probe, which finds the strand and removes it. The `{}` Status then reaches Wait's liveness tick. It asserts `OutcomeDied`, a `stopped` record, and one `RemoveStrand` call. The sequence is consistent. I assumed `fastConfig` has `LivenessEveryNPolls <= 1` and that the fake reed holds its last queue entry. Neither is in my context.

**Findings**

- **LOW, design.** `wait.go:1270` and `stop.go:40`: `stopMarked` is set before the removal runs, and is not cleared when removal fails or does nothing.
  - **Removal fails or does nothing.** `removeStrandIfPresent` can fail on a Status or `RemoveStrand` error. It also does nothing for an untracked or non-removable strand, a window-0 dead pane for example (`stop.go:100-113`). Either way the handle stays marked.
  - **Strand later lost for another reason.** If reed's table is later reset, for example by a `reed down` or a lost `reed.json`, Wait now answers `OutcomeDied` and records `stopped`, where before it failed with `errStrandNotTracked`.
  - **Why it is arguably acceptable.** The caller asked for a stop and the record already reads `stopped`, so the answer is not wrong. It is a masking window the summary does not mention, and it matters for the duplicate-agent hazard `errStrandNotTracked` documents (`wait.go:162-171`). Nothing pins it with a test or a doc line.

No other defects in this lens.

**Tool calls (1)**
- `grep recordMu` over `internal/**/*.go` (non-test): my context lacked the full set of lock sites, which I needed to settle deadlock and race.
