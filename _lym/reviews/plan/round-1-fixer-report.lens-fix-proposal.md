```yaml
lens: fix-proposal
outcome: reported
```

I checked the fixes made as the review proposed, looking for cases where the proposal itself is wrong.

1. MEDIUM — card 15, the new exempt-paths sentences: "The exempt paths are those loom itself writes on the weft during a row: `judgement.yaml` and everything under `RunRecordsDir`…" and "A moved weft HEAD counts as a change only when `WeftChangedSince` … names a path outside the exempt ones".
   Summary: the exempt set leaves out the friction directory `_lym/loom/friction/`, which loom also commits on a resume. So a resumed Drift-Verify with an open judge mark can still stop on loom's own commit, which is what F1 says.
   Evidence: `docs/overview.md`, the drift tail paragraph (around lines 608-612) says "Every `lym loom start` or `step` commits whatever the friction directory holds along with the seed". The same paragraph says the friction directory holds "the crash-resume notes written at a `run` or `step` entry". The case: the judge is interrupted, `lym loom start` writes a crash-resume note and commits it on the weft, the weft HEAD moves, and `WeftChangedSince` names `_lym/loom/friction/&lt;note&gt;.md`. That path is outside `RunRecordsDir` and is not `judgement.yaml`, so step 5 stops on the judge-change Stuck. The new TestJudgeSnapshot row and the new TestDriftVerify row cover only a commit of `status.json`. A loom resume commits more than that.

2. LOW — card 8, `CheckBaseline` now faults "for a missing baseline directory whether or not a commit carries the trailer". Card 19's `Defer` admits a blocked writer whose "baseline check `CheckBaseline` fails when it runs".
   Summary: on a scope-`off` run, no baseline directory ever exists. So `CheckBaseline` faults, and `lym drift defer` is admitted at any blocked Darn-Write or Webster halt, even one that has nothing to do with drift.
   Evidence: before the fix, `Unguarded` "checks nothing", so a run with no `Drift-Baseline` commit never admitted Defer this way. Now a scope-`off` run blocked at Darn-Write on an exhausted verify gate admits `lym drift defer` and records `deferred`. That widens D12's admission beyond the halts it names. Card 19's admission rule does not exclude scope `off` or an absent scope record before it calls `CheckBaseline`.

3. LOW — card 19, Defer's new held-lock way forward: "waiting for the running capture or check to finish, then running `lym drift defer` again".
   Summary: when the lock holder is the main capture at a blocked Drift-Await, the old way forward was `lym loom resume`, whose row waits on the running capture. The new text drops it. Once that capture passes, Drift-Await counts it on resume, so a defer is no longer needed. Telling the caller to run defer again can defer a capture that would have counted.
   Evidence: today's refusal in `internal/driftengine/verbs.go:233` reads "lym loom resume, whose row waits on the running capture". The new wording names only the defer re-run, for both kinds of lock holder.

No findings on F2, F3, F4, F5, F7, F8, F9 or F11. CountedCapture is named by no card other than card 25, so deleting it there breaks nothing.

Tool calls:
- grep of `_lym/plan` for `CountedCapture` and for `friction` in cards 15 and 16. My context did not show whether any card between 16 and 25 still uses CountedCapture, or whether the judge cards mention the friction directory.
