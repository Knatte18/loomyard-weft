```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fixer carried out as proposed (F1, F2, F3).

no findings

- **F1:** the wait is registered before `tmuxkit.KillOnCleanup` and the teardown closure, so it runs after both kills. The temp-dir removal was registered earlier, by the fixture's hub copy, so it runs last. Dropping the explicit call in `TestSmokeWatchdog_StartSpawnsTheDaemon` loses nothing. I built no case where the change is wrong.
- **F2:** the proposal removes the old fixed ceiling of two patiences, so the loop would not end if a surviving spawner showed a new pid at every deadline scan. That needs a spawner outside the temp root that the teardown does not kill, and I found none in the code in context. I am noting it, not raising it.
- **F3:** the `//testtiming:keep` marker follows the repository's form and names what it pins.

Tool calls made: none.
