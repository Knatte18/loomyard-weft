---
round: 3
ledger:
  - key: comment-hard-wrap-multi-sentence
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: BLOCKING
  - key: watcher-stopping-erased-by-tick-save
    rounds: [1]
    status: resolved
  - key: orch-reload-stencil-comment-stale
    rounds: [1]
    status: resolved
  - key: untagged-tests-missing-parallel
    rounds: [2, 3]
    status: open
    class: consistency
    severity: BLOCKING
  - key: reset-unreachable-fallback
    rounds: [2]
    status: resolved
  - key: test-names-contradict-assertions
    rounds: [2]
    status: resolved
  - key: start-watchdog-test-stale-gate-comment
    rounds: [2]
    status: resolved
  - key: watcher-run-done-shadowed
    rounds: [2]
    status: resolved
---

Round 1 raised three findings; round 2 resolved two of them and reopened the comment-line-break key.
Round 3's fresh review found none of the round 2 design, scope or naming defects (the unreachable reset fallback, the contradicting test names, the stale gate comment, the shadowed `done`), so those four keys are resolved.

The comment-line-break key is open for a third round.
F1 covers new lines hard-wrapped mid-phrase in `completionsignal_enforcement_test.go`.
F2 and F3 cover added comment lines that join two independent clauses with a semicolon, or with a comma and a conjunction, across many files.
The review's lists come from a line scan, so the fix is a sweep of every added or changed comment.

The missing-`t.Parallel` key is open for a second round.
F4 names `TestWatcher_AutoCompactionReloadsPluginsThenPointer` and `TestCompact_CycleReloadsPluginsThenPointerNamingTheNote`; F5 lists substantially edited untagged tests that stay serial with no named global state.
Both keys are now class `consistency` in the latest review.
