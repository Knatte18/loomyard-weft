```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged the parts of the review's proposals that the fixer carried out as proposed (F1's skips, pass-through cases and hold behaviour; F2's bound and criterion; F3's criteria).

Proposals fixed as proposed and challenged: F1 parts (2), (3) and (4); F2; F3.

1. **BLOCKING.** Changed text: D4, "The push goes unchecked, as a loom run's does, when pushes or git are off (`fabricengine.EnvSyncOptions`) or a pause covers the run."
   - Summary: skipping on a pause makes the hand verb push exactly when loom's own stop has frozen the run.
   - Evidence: `prePushCheck.guard` in `internal/loomcli/prepush.go` pauses the run when it stops a push. The cases are an entry another machine owns (`pauseOwned`), an entry origin ended whose parent lacks the work, and a stopped branch.
   - Loom can skip on a pause because a paused loom run does not drive its own pushes. A hand `lym gate testmap` call has no such reason.
   - Concrete case: loom stops a task's push because machine B owns the entry, and pauses the run. The operator then runs `lym gate testmap` by hand in that task. `pauseCovered` reads true, the verb skips the check, and it pushes the task's code branch over machine B's ownership.
   - That is the routing edge the reconcile was added to close. It also contradicts the criterion "a testmap call in a task whose entry the run-scope origin reconcile reads as ended … pushes nothing", whenever a pause covers that run.
   - The verb should not take the pause skip. It should judge a paused run like any other.

2. **MEDIUM.** Changed text: D4, "It goes unchecked too when … its run stands at Finalize, which judges origin itself."
   - Summary: Finalize judges origin for its own commits and pushes, not for a separate hand call's push.
   - Evidence: the `skipsCheck` doc says Finalize "judges origin and its owners itself". That judgement belongs to Finalize's own steps, so loom's other pushes can skip.
   - Concrete case: a hand testmap call made while a task's run stands at Finalize. Origin has ended the entry, or another machine has taken it. The verb pushes the code branch with no check at all, because Finalize's check never runs on that push.
   - The verb should not take the Finalize skip either. Together with finding 1, of loom's skips only "pushes or git off" and "the prime" carry over to a hand call.

3. **LOW.** Changed text: the two new D4 criteria, "a testmap call in a task pair that changed maps spawns the detached records push, and that push carries the task's code branch" and "a testmap call in the prime that changed maps spawns a records-only push".
   - Summary: both are stated unconditionally, so they contradict D4's own exceptions.
   - Evidence: D4 says no push is spawned on a hold, and the existing held-push criterion says the call "pushes nothing". Under `EnvSyncOptions` with `SkipPush` or `SkipGit` there is also no push to observe.
   - Each criterion should be conditioned on pushes being on and, for the task pair, on the verdict letting the push go.

4. **LOW.** Changed text: D5's new criterion, "a map rewritten between the pre-pull commit and the replay's dirtiness check stops the pull with `ErrReplayDirty` …".
   - Summary: a test can only reach this window deterministically through a seam between the pre-pull commit and `replayRequest`'s check, and D5 does not name one.
   - Evidence: the window is a race against a live pass, and the testing rule says a test must not depend on timing.
   - Without a seam, the plan writer will either write a flaky test or drop the criterion. The criterion could say the rewrite is injected at that point.

The F2 bound itself holds: `replayWeft` tags set-aside commits before `cherryPickOnto` refuses, so "leaves the weft branch unmoved" is accurate.

No further tool calls made.
