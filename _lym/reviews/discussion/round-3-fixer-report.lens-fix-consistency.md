```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 change set agrees with the decision record and support log around it (fix-consistency lens).

**Finding 1 — LOW**
- **Changed text:** `_lym/discussion/decision-record.md`, Decision 5. Two sentences are involved: the rescoped "Only the holder of a file's own dobby lock rotates `dobby.log`" and the new Windows sentence "the exception to the rule that only the lock holder rotates".
- **Summary:** The only stated rule is now scoped to `dobby.log`, but the Windows spawner rotates `dobby.out`. Calling that an exception to the lock-holder rule points at a rule the record no longer states for `dobby.out`.
- **Evidence:**
  - After the fix, the record states the lock-holder rule for `dobby.log` only.
  - For `dobby.out` on exec platforms, the only text is the sentence "There the process that takes the lock, at each start, fresh or re-exec, rotates `dobby.out` by the same rule". It describes what happens and states no only-the-holder rule.
  - So "the exception to the rule that only the lock holder rotates" has no `dobby.out` rule to be an exception to.
  - A reader can still reconstruct the intent. The fix would be to state the rule once for both files: `dobby.log`, plus `dobby.out` where `selfexec` execs.

I checked these siblings and found them consistent:
- **Decision 1:** the exit line, its bound and its tidy stop agree with Decision 3's exit list, the "no append creates a directory" reference in Decision 3, Decision 4's tidy stop, and both acceptance criteria.
- **Decision 2:** the re-exec test, named once, agrees with its own Bound on the second read, with Decision 3's gated notices and with the acceptance criterion on records without start ticks.
- **Support log:** the round-3 entry matches the record.

Further tool calls: none.
