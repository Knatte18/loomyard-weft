```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's F8 change set agrees with its surroundings (fix-consistency lens).

- LOW: the D7 criterion "D7: a walk whose board write fails leaves the entry as it was and the build stamp stale, and the next start verb's walk carries it" (decision-record line 226) was left as it was. The fix changed its twin in D7's body to "the next walk retries" and made D5 run the walk from every verb that loads the loom or tapestry config. The criterion still names only a start verb. It is not false, but it now disagrees with D7's own sentence and with D5. Evidence: D7's body says "the next walk retries", and D5 line 117 says "Every lym verb that loads the loom or tapestry config runs `hubreconcile.EnsureOrWarn`".
- LOW: the new D5 line 117 says "so the first verb of any kind on a new build runs the walk". The same sentence's subject limits the call to verbs that load the loom or tapestry config, so "of any kind" overstates its own rule. Bound line 2 says the same thing correctly: "at the first verb that loads the loom or tapestry config". Evidence: line 117 and the D5 Bound line that follows it.

Calls made:
- grep for "start verb" and "before any command" in the decision record and support log: I needed to see whether any sibling text or support-log entry still states the old premise. My context does not hold the support log, or the record after its edits.
