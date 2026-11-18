# Case brief: run `shuttle-start-idle-and-prompt`, Land producer refused on an ended origin entry

## 1. The question the next judge must answer

The stop is a `producer` error at `Land` (history length 7, progress "failed at Land"). The error is that the records push of `main-weft` was refused because origin ended the board entry of that branch. Is the run's `Land` step safe to resume, or does the ended entry mean the run must be handed back or landed by another path? Which of these is right, and is a re-run of `Land` going to hit the same refusal?

## 2. The facts, each with a pointer

- Stop envelope: `/home/knatte/Code/loomyard-LYMHUB/loomyard/_lym/shed/shuttle-start-idle-and-prompt/drive-reports/drive-envelope-20261118-110920-8bb6.json` line 1. `kind: producer`, `transient: ""`, `loop.current_producer: Land`, `interrupt_policy: handback`, `hand_back: false`, `state: failed`.
- Error text, from the same envelope and from the stop step's trace copy `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/shed/shuttle-start-idle-and-prompt/steps/b8f7fbdbc337191a.trace.log` line 6: `the entry of branch main-weft is ended on origin, so the branch was not pushed and is left to its teardown`.
- Trace `b8f7fbdbc337191a` line 5 (`fabric: mutation`, `kind=commit_created`): the weft commit `aa13796e3bc4b6a61e995f28a70849038e4853c7` was created at 11:09:17.639 on `/home/knatte/Code/loomyard-LYMHUB/loomyard-weft`.
- Trace `b8f7fbdbc337191a` line 3 and the loop log: `fabricengine: a run origin has ended is left to this machine's own landing` (11:09:17.606). The same info line appears in earlier steps `493a3dd5501dab3e`, `3af7cb55ca19c172`, `37c828e0a7656e65` (loop log `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/shed/shuttle-start-idle-and-prompt/steps/loop.log`).
- Loop log history: steps through 11:08:58 sit at `Run-Shed` step 6 of 8, then `76c819214103dfa4` reports `child running → done at Friction-Reflect` (history 6), then the `b8f7...` step at 11:09:16 fails at `Land`.
- Driver log `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/logs/trace-20261118T100915Z-80dfbc13a3f5a518-1547798.log` line 4: tackler spawned for attempt 2, producer `Land`.
- Remote tip (read with `git ls-remote origin main-weft`, run in `/home/knatte/Code/loomyard-LYMHUB/loomyard-weft`): `15c0ddfb2eb8167895e02804e2b91534243d11d4 refs/heads/main-weft`. The branch is present on origin, so it is not absent.
- Local `main-weft` (`rev-parse --verify`) is also `15c0ddfb2eb8167895e02804e2b91534243d11d4`. Local and origin agree, so no unpushed commits are left on the local branch.
- `git merge-base --is-ancestor aa13796e... main-weft` returns true. The commit the stop names is an ancestor of origin's tip, so by the rule in the prompt it is present on origin. It is not absent.
- Code that emits the error: `/home/knatte/Code/loomyard-LYMHUB/loomyard/internal/fabricengine/pushanchored.go` lines 297-310 (`recordsSideNotLanded`). It returns the ended-entry error when push returned nil, and no deferred failed-push record was found for the branch. Note the function is reached after a push returned nil.
- Ended-entry check: `/home/knatte/Code/loomyard-LYMHUB/loomyard/internal/fabricengine/pushclassified.go` lines 494-507 (`checkOrigin`, which says the result is nil for a branch whose entry origin has ended) and lines 629-664 (`goneEntryEnded`, which fetches the Bolt board branch and reads the ended set through `ledgerRunsAt` at the board's head).
- Stop step stderr `/home/knatte/Code/loomyard-LYMHUB/loomyard/.lym/shed/shuttle-start-idle-and-prompt/steps/b8f7fbdbc337191a.stderr.log` line 1: the same refusal, nothing more.
- Loop keeper exit: loop log last lines, `loop ce69f7fc219fa0b2 keeper process 1547831 exited: exit status 1`. This may be a symptom of the stop and was not examined further.

## 3. What you ruled out, and how

- A missing remote branch as the cause. `git ls-remote origin main-weft` answers a line, so the branch is present. Ruled out.
- An unpushed weft commit being stranded locally. Local `main-weft` equals origin's tip, and `aa13796e` is an ancestor of that tip. Ruled out as the reason the branch was "not pushed" in git terms.
- A transient error. The envelope has `transient: ""`, and the error is a fixed string from `recordsSideNotLanded`, not a transport failure. Ruled out.
- A plain re-launch settling it. The ended-entry check reads the board state on origin, which the re-run would read again. A re-run of `Land` is therefore expected to hit the same check. This is an expectation from the code path, not a tested result.
- A `goto` to an earlier row. The stop is at `Land` and the commits already exist on origin, so an earlier row would redo work without changing the ended entry. Not chosen.

## 4. Your hypothesis, marked as a hypothesis

HYPOTHESIS (not verified): The run's board entry on origin was ended by another machine or by a prior landing of the same run (the repeated `a run origin has ended is left to this machine's own landing` lines suggest origin considers the run ended and expects this machine to land it). The `Land` push then correctly refuses to push a branch whose entry is ended, and the branch's commit reached origin anyway (it is an ancestor of origin's tip). If so, the right move is for the run's parent or a stronger judge to decide whether the run should be marked landed/teardown-only, not a re-launch. The code read needed to confirm this: the ledger contents read by `ledgerRunsAt` in `goneEntryEnded` at the board's head, and whether `aa13796e` was pushed by this run or was already on origin before 11:09:17.
