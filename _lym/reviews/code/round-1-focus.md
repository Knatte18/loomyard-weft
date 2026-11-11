---
round: 1
focus:
  - "internal/raddleparser/check.go and internal/boardengine/store.go: does the note-dependency refusal fire at propose time, name the way forward (promote the note or drop the dependency), and complete the Raddle-File refusal text with the missing command?"
  - "internal/shedverbs/loopwait.go and loop.go: for a loop that ends without an envelope, does the halt note and run status carry the exit code or signal, and when it cannot be captured, the reason instead of 'not recorded'? Check the part commit d1535412 already landed was not redone."
  - "internal/shedverbs/loopwait.go: does the 'loop log' field show the tail of the interrupted loop's own output rather than earlier steps' envelopes, including empty-output and very long-output edge cases?"
  - "contracts/specs/raddle-proposal-spec.md and internal/shedverbs/doc.go: does the changed doc text match the behavior the diff implements?"
  - "Tests in check_test.go, store_test.go, loop_test.go, loopwait_test.go: does each new or changed behavior have coverage, and does any new test duplicate an existing one?"
---

Seed focus from the board entry's two notes (`raddle-propose-note-deps`, `loop-exit-log-tail`) and the diff stat.
