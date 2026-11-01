```yaml
lens: fix-consistency
outcome: reported
```

Task: check through the fix-consistency lens that the round-2 changes agree with the sibling decisions, criteria and cross-references they touch.

1. **MEDIUM.** D1's new sentence "A front's cap counts from the front's own start, and its lease deadline (D3) is that start plus the cap" disagrees with D3.
   - Summary: D1 now defines the lease deadline as start plus cap, but D3 still says each front writes "its own cap deadline plus a grace of 120 s".
   - Evidence: D3's first paragraph reads "Each front writes the lease … its own cap deadline plus a grace of 120 s, to cover the agent's next turn", and D3's Bound reads "at most the long cap (D5) plus the grace".
   - A reader of D1 gets a lease that ends at the cap, so a runner would be abandoned just as its front answers 75, before the agent's re-run.
   - D1 should say the cap deadline is the start plus the cap, and that D3 adds the grace to it.

2. **LOW.** D6's new paragraph has a twin missing in D1. D6 says "a front that supersedes a dead runner (D1) kills that group through `proc.KillTree`", but D1's supersede sentence was left as it was.
   - Summary: D1's supersede path still describes only the lease expiry and a fresh start, so the kill of the dead card-verify runner's recorded line group appears only in D6, which points back to D1.
   - Evidence: D1 reads "Otherwise it expires the current run's lease, so a live runner ends at its next lease read, and starts a fresh runner under a new run id." It says nothing of a dead runner's recorded process group.
   - D1's acceptance criteria also carry no supersede-kill row; only D6's criterion does.

3. **LOW.** D1's new lock wait ("A front waits for the key directory's lock only while at least 30 s of its cap remain") is a new timed wait, but the PATTERN-daemon-wakeups Constraint was left as it was.
   - Summary: the Constraint lists only "the front's wait, its claim wait and the runner's lease read" as waking at most once a second. The deadline-bounded lock wait, a new wakeup source, is not covered.
   - Evidence: the Constraints line reads "PATTERN-daemon-wakeups: the front's wait, its claim wait and the runner's lease read wake at most once a second."

Further tool calls: none.
