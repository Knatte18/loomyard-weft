---
round: 1
focus:
  - "Re-judge inputs: `listFiles` in `internal/driftengine/loosened.go` drops only the stored rulings that hold an uncaused check. Settle whether the judge spawned afterwards can learn that the dropped checks now live in another test, including when that test file's own ruling is still reused and so is not listed. Read the judge's stencil and request: can it rule such a file anything but `uncaused` without a trailer?"
  - "Both callers of `listFiles` and `newRulings` in `internal/driftengine/verify.go` (the capture path and the recorded `command` diff path): settle whether each re-judges and halts as the amended behaviour says, including the early returns before the file listing, such as an unchanged head."
  - "Wording of the way forward: compare `uncausedReason`, its doc comment, the package doc and `rangeRulingKey`'s comment with what the key hashes (file diffs and trailer lines, not commits). Settle whether any claim that a new commit triggers a re-judge holds for every such commit."
  - "Docs outside the package: settle whether `docs/overview.md`, where it describes the ways out of an uncaused ruling, and any other doc or stencil naming those ways, still agrees with the diff."
  - "Merge danger: the change description calls `two-way` while `JudgedRecord` in `judged.json` gains `range_key`. Settle whether another binary version reads, rewrites or guards that record in a way that makes the call or the stated blast radius wrong."
  - "Test coverage: the mutation report lists a surviving mutant in `rangeRulingKey`. Settle which input of the range key no test pins, and whether a test pins that a ruling with no uncaused check is still reused when another file changes."
---
Seeded from the committed diff, the change description and the commit messages.
The entries are questions to settle, not findings.
