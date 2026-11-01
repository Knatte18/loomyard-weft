```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 and F2 proposals as the fixer carried them out.

no findings

- **F1:** The three added exception sentences match the code the review cites: `checkLivenessTick` returns `OutcomeDone` on `allOutputFilesExist` before the `markedStopped` branch (`wait.go:1268-1271`). So "recorded as stopped unless the files show the run done" is accurate.
- **F2:** With the output file present, the `OutputPresent` row has no events file, so `Wait` reaches the liveness path.
  - The earlier `allOutputFilesExist` returns in the events path (`wait.go:887`, `894`, `904`) sit behind `len(events) == 0 → expiredTurnEnd()` (`wait.go:877-878`).
  - The `awaitStartup` returns (`wait.go:677`, `717`) belong to `Start`, not `Wait`.
  - So the row exercises `checkLivenessTick` itself. Swapping the stop branch above the file check would make it return `OutcomeDied` and fail the row.

I did not check whether `expiredTurnEnd` itself returns `OutcomeDone` on an empty events read.

Calls made:
1. `grep` on `wait.go` for `allOutputFilesExist` and `markedStopped`: to find any earlier file-check return in `Wait` that would let the new done row pass without pinning the order. My context lacked the file's body beyond the header and the `Wait` doc comment.
2. `Read` of `wait.go:640-730`: to rule out the `awaitStartup` hits as `Wait` paths. My context lacked that region.
3. `Read` of `wait.go:872-912`: to see what the events-path returns need before they fire. My context lacked that region.
