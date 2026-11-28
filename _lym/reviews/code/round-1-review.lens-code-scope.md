```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the amended board entry's goal points.

**Decisions (decision record alone; the record has no `Added after Discussion` entries)**
- G1: "Make the lock-free pass gate against the parent's local head when that head is ahead of origin only by owed landing tips" (`_lym/discussion/decision-record.md:20`).

**Per change**
- C1: the `takeOwedLocal` flag and the `ParentAhead` branch in `parentMergeTip` (`internal/landingshed/parentalign.go:232-258`). They take the local tip when `ParentAheadByOwed` answers true, and turn a failed read into an error. This traces to G1; the error path comes with the new seam read.
- C2: `Finalize.gateUnlocked` passes `true`, and its doc comment gains two lines (`internal/landingshed/finalize.go:360-361`, `finalize.go:380`). This traces to G1.
- C3: `ParentAlign.Call` passes `false` (`internal/landingshed/parentalign.go:147`). C1's signature change forces it, and it leaves Parent-Align's behaviour as it was, which G1 does not ask to change. Forced by C1.
- C4: a new sentence in the package doc, `internal/landingshed/doc.go:42`. It is a doc that C1 and C2 would leave stale. Forced by C1 and C2.
- C5: the `owedAhead` row and field in `TestFinalize_GatesWithNoLockAndLocksToLand`, and the failed-read row in `TestFinalize_LockFreePassStops` (`internal/landingshed/finalize_unlocked_test.go:47-96`, `finalize_unlocked_test.go:280-289`). They are tests of C1 and C2. Forced by C1 and C2.
- C6: commit `31f28dbee`, which rewraps the new comments at sentence boundaries. It is a formatting edit of the comments C1, C2 and C4 add. Forced by C1, C2 and C4.

**Per decision**
- G1 is built by C1 and C2, with C3 to C6 forced by them.

**Findings**
None. Every change traces to G1 or is forced by a traced change, and G1 is built.

Tool calls: none.
