```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: does the fix's change set agree with the sibling decisions, criteria and cross-references it touches?

Findings:

1. **MEDIUM.** D8 Bound, "the old owner's unpushed work reaches the adopter only through D4's archive tags", against the new D4 and D6 step 4 text.
   - **Summary:** The fix contradicts this untouched Bound. The old owner's code commits can now reach the adopter on the code branch itself, not only through the archive tags.
   - **Evidence:** D4's changed hand-over bullet says the old owner "pushes a code tip that a transition already on origin names and origin's code branch lacks". D6 step 4 says it "pushes a code tip a transition on origin names and origin lacks (D4)". Both pushes happen after the takeover, so code that was unpushed at takeover time lands on the branch. The D8 Bound's "only through D4's archive tags" was left as it was.

2. **MEDIUM.** D8 step 3: "A code tip the last transition names that origin still lacks is fast-forwarded to when it lands, before the act rule lets the run act (D4)."
   - **Summary:** This names a step no process can carry out. `lym adopt` runs in the foreground and exits after step 8, and D4's twin text names no actor for a fast-forward after adopt ends.
   - **Evidence:** D8 says "`lym adopt <subject>...` runs every step in the caller's process" and starts no detached worker. D4's new bullet says only that "the act rule answers `wait`" and that the owing machine "pushes it". Nothing in D4, D6 or D1 says who fast-forwards the adopter's local code branch once the tip lands later. The new criterion "D4: … waits until that tip lands, then goes on from it" depends on that unnamed step.

3. **LOW.** The D5 clash refusal's new way forward, `lym machine name --forget-former <name>`.
   - **Summary:** The fix's twin in D8 was left as it was, and D5 never introduces the new flag.
   - **Evidence:** The review's issue text says "D5's refusal also fails D8 step 1's owner write". D8 step 1 still lists only the refusals over another machine's hold or owner, with no clash refusal or its way forward. D5's third bullet ("`lym machine name` records the rename there…") and the Constraints' `PATTERN-shell-guard-table` and `PATTERN-refusal-way-forward` lines don't mention the new `--forget-former` flag outside the refusal sentence and its criterion.

4. **LOW.** support-log.md, Turn 5 lines describing D1, D6, D9 and D10 as fixed.
   - **Summary:** These lines now describe the record wrongly.
   - **Evidence:** "fixed, D1's act rule, D6 step 4 and D9 carve it out" is no longer true: the carve-out moved to the owner (D1's `hand_over` line, D6 step 4's "a `landing` past its parent-side merge included", D9's "ends here too"). "origin's wins when another machine wrote it" is also stale: D10 now restores by the session record's last launcher or resumer, and "whatever machine the mirror names" the local transcript wins. Only Q5 in the log was updated.

5. **LOW.** D1's line "A pair stands or not beside each state; a landed or cancelled run whose pair stands owes a teardown (D11)" against the new D11 first bullet.
   - **Summary:** The D11 condition was narrowed, but its D1 twin still says every landed or cancelled run with a standing pair owes a teardown.
   - **Evidence:** D11 now tears down only "`landed`, or `cancelled` without keep, with no pause or hold covering it and no other machine owning it". D1's sentence still says every such run owes a teardown, which conflicts with D11 for a kept cancel or a run owned elsewhere.

Further tool calls: none.
