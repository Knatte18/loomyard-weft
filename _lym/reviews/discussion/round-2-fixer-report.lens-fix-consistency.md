```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: checking whether the change set agrees with the sibling decisions, criteria and cross-references it touches, and whether any change left its twin elsewhere unchanged.

1. **[MEDIUM] The new D8 Bound contradicts the PATTERN-child-processes Constraint.**
   - Changed text: D8 Bound, "a call naming no package and lacking `--upkeep` is not registered, so it can run beside a pass".
   - Evidence: D9's last bullet says such a call "runs as a pass". The Constraints section says "PATTERN-child-processes: a pass stays registered as `runproc.RoleTestMap`".
   - The record now says both that every pass stays registered and that one kind of pass is not registered. Either the Constraint should be narrowed to a dobby-started or `--upkeep` pass, or the Bound reworded.

2. **[LOW] The new D8 criterion asks for a live child that cannot exist.**
   - Changed text: Acceptance criteria, "a `--upkeep` call whose registration fails exits before it plans, and the dobby starts no pass while that child lives".
   - Evidence: The first half has the child exit before it plans, so the second half has no live child to test against. D8's second bullet states the live-child rule on its own, as a guard for the time before registration, but the criterion ties it to a child that has already exited.
   - The second half should be its own case, such as a started child that has not yet registered.

3. **[LOW] Support-log Turn 5 still describes a parent question under Open risks.**
   - Changed text: The Q15 resolution and the dropped Open risks bullet. The fix updated its twin on the board-scope line but not this one.
   - Evidence: support-log Turn 5, the bound-breaker/walkthrough line, still reads "T4's running status stays a parent question under Open risks." No Open risks line holds that question now, and Q15 is resolved.

Out of my lens, one sentence: a `--upkeep` call that exits before it plans writes no exit record, so D8's trigger conditions still hold on the next tick and the dobby may restart it every tick while registration keeps failing.

Tool calls: none.
