```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal, stop and recovery step the cards add, I checked whether the next step it names can succeed from the state it leaves.

**BLOCKING — card 5 capture-run, with cards 7, 9 and 10 (the capture start).** `StartCapture` can return before the capture child has passed its own precondition check, so a capture can silently never run.
- Card 5's order is: `RunCapture` takes `CaptureLock`, *then* runs `Check` (`CheckMainCapture` re-reading the status), *then* writes the `running` record.
- `StartCapture` returns "once `ProbeLock(CaptureLock)` reads held", which can be before `Check` has run.
- Loom row start: card 9's `Drift-Baseline` returns Done at that point and the run moves on to `Plan-Write`. If the child's `Check` then reads `Plan-Write` as the current row, it refuses. That refusal writes no record, so nothing captures. Card 5 admits only `Drift-Baseline`, or `Drift-Await` while `blocked`.
- Caller start: card 10's `Drift-Await` halt names `lyx drift capture` followed by `lyx loom resume`. Card 7's `capture` returns once the lock is held. A prompt resume sets the status to `running` at `Drift-Await`, which `CheckMainCapture` does not admit. The child exits with no record, and the resumed `Drift-Await` halts again on the old missing or failed record.
- In both cases the named next step does not progress.
- Fix: return from `StartCapture` only once a fresh `running` record exists, which is written after `Check` passes. Or run the precondition in the spawner and have the child skip the row check.

**MEDIUM — card 11 drift-verify-producer, step 8 and step 6 (resume after a dirty-tree or test-gate Stuck).** This halt names no next step, and a plain resume from it skips what the gate guarded.
- The Stuck after the `dirty` or `verify` entry spent its attempts "keeps the gate's reason and findings file". It names no way forward such as committing or cleaning the tree, or fixing the tests, then `lyx loom resume`.
- On resume, `Call` goes straight to step 6, the mechanical compare. Nothing re-checks the tree or re-runs the `drift-verify` test gate.
- A dirty tree is compared as it stands, and a pass returns Done with uncommitted changes.
- A test-gate failure is passed by on a passing compare.
- So the resume reaches Done without lifting what the halt was about. D6 orders dirty tree, then verify, then compare at every way out.
- Fix: name the way forward in these reasons. On entry, refuse to compare a dirty tree, and re-run the verify gate when HEAD differs from its last pass.

**MEDIUM — card 5 `CheckMainCapture` and card 9 `Drift-Baseline` (the dirty-tree capture halt).** One of the two named ways forward cannot lead to a capture.
- The dirty-tree refusal names "committing or cleaning the listed paths and `lyx loom resume`".
- Committing moves HEAD off the branch point, the merge base that `pairMergeBase` reads in card 14.
- The resume then fails `CheckMainCapture`'s HEAD rule and halts naming only `lyx drift defer`.
- Cleaning works. Committing never captures again, although D3 says the resume "captures again".
- Fix: name only cleaning or stashing the listed paths, or say that a commit leaves `lyx drift defer` as the only route.

**LOW — card 5 `StartCapture` (a child that exits first).** The error names a log directory that holds nothing about why the child exited.
- `StartCapture` errors "naming the capture log directory" when the child exits before taking the lock.
- The child's own refusals (lock contention, `Check`) happen before `RunCapture` opens its log.
- The detached child has no stdout or stderr handle, so the reason is lost.
- Fix: give the spawn a log for the child's own output and name that file.

**LOW — card 9 `Drift-Baseline` (a `StartCapture` failure).** The card never says what a `StartCapture` failure returns.
- `ErrBreakawayRefused` is covered, but a child that exits first and a spawn that fails to start are not.
- The card should state that these return an error, so the stop is resumable. Then the resumed entry captures again, since the lock is free and no record passed.

**LOW — card 7 `lyx drift compare` and `lyx drift capture --side` (`TakeRun` at a halt).** The budget refusal tells the wrong caller what to do.
- These verbs check only that the current row is `Drift-Verify`, so a parent at a halt can run them and spend the agent's counter.
- Card 2's refusal text, "write what you found to verify-report.md and end your turn", is a way forward for the agent session only, not for a parent at a halt.
- Fix: refuse these verbs outside a `running` status, or word the refusal per caller.

I made no further tool calls; every finding comes from the plan and the exploration already in context.
