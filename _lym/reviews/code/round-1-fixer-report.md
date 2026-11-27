---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The door is now one-way. The blast radius stays `webster`, because mergeDangerFindings in internal/summaryparser/description.go rejects a blast radius of more than one word, so the key rename is stated as prose below it. Only the older-binary direction is described, since the F2 fix removes the other."
  - id: F2
    disposition: fixed-differently
    reason: "The legacy path is appended to MasterNotePaths, not prepended. When both keys are present, an older binary wrote the single key after the list, so prepending would break spawn order. With an empty list both give the same result."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The proposed sentence also says the list is empty for a state written before the field existed, which is false once F2 loads the old key into it. The doc now says only that the list is empty when friction notes were off at every Master spawn."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

## Changes

- F1: `_lym/landing/summary.md` marks the door one-way and describes the key rename under `## Merge danger`.
- F2: `State.MasterNotePath` (`masterNotePath`) is read by `LoadState`, moved into `MasterNotePaths` and cleared.
  A `TestRun_DoneOutcome` row covers the resume and checks that the saved file drops the key.
  The summary's Changes and Design decisions sections match.
  Fixed differently: appended rather than prepended.
- F3: the `ClassParentWrite` comment in `audit.go` names every note a Master prompt of the run named.
  The `runlevel.go` comment on the verify-fix prompt, which made the same single-note claim, is corrected with it.
- F4: the `MasterNotePaths` doc is corrected.
  Fixed differently: the clause about older states is dropped.
- F5: two `TestRun_DoneOutcome` rows cover the already-listed path and the resume with friction notes off.
- F6: the parent-write violation `Detail` names the friction notes the run's Master prompts named.

A gate finding on the round's own change is repaired: the `ClassParentWrite` comment had a line wrapped mid-sentence.
Its first sentence is now one shorter line.

`lym gate test ./internal/websterengine` passed after that repair (0 failed).

## Self-check

- fix-code-correctness, LOW, prepend breaks spawn order with both keys: fix with own change (append).
- fix-code-correctness, LOW, no test pins the key clear or the load-time duplicate skip: fix as proposed (the legacy-key row reads the saved file, and the already-listed row also seeds the single key).
- fix-consistency, LOW, summary Changes bullet reads as if the old field were gone: fix as proposed.
- fix-consistency, LOW, prepend breaks spawn order: duplicate of the first, fixed.
- fix-proposal, LOW, prepend breaks spawn order and the `MasterNotePath` doc misses the older-binary source: fix with own change (append, doc reworded).
- fix-affected-set, LOW, summary Changes bullet stale: duplicate of the fix-consistency one, fixed.
- fix-affected-set, outside its lens, prepend breaks spawn order: duplicate of the first, fixed.

## Disputed

## Deferred

## Rulings
