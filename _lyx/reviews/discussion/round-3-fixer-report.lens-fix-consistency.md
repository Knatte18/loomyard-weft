```yaml
lens: fix-consistency
outcome: reported
```

I checked the fix set against the sibling decisions, criteria and cross-references it touches or depends on. Three findings.

1. **MEDIUM.** Changed text: D10 step 4, "uncommitted weft content outside that pathspec refuses the step without force … naming the paths and commit or discard in the weft worktree as the way forward".
   - Summary: this adds a new refusal to `promote`, but its twin places in the record were not updated.
   - Evidence: D10's refusal list (line 163, "It refuses, each with its way forward: …") does not include it.
   - Evidence: the D10 acceptance criterion (line 290) lists five refusals and not this one, but the criteria section opens "Every refusal below is reached with its way forward in its text", and the Constraints bullet PATTERN-refusal-way-forward says "every new refusal names its way forward, and a test reaches it".
   - Evidence: "without force" points at a force that D10 never gives `lyx beam promote`: no `--force` flag is named in D10, D8 or the Scope. So the "force" stage of the pipeline that step 4 cites has no caller input.

2. **LOW.** Changed text: D13 lines 223-224, "The Land row's ready notice takes no walk".
   - Summary: the Scope bullet that mirrors D13 now overstates it.
   - Evidence: line 19 still says "Notices and parent names go to the nearest live orch up the beam chain", with no exception. D13's own heading at line 217 says the same, "Parents and notices go to the nearest live orch up the chain".
   - Evidence: line 221, "The notice and parent-name readers take the walk", is followed two lines later by the ready-notice exception. That reads acceptably inside D13, but the Scope line is now a twin that was left as it was.

3. **LOW.** Changed text: D10 Bound, records "stay reachable only in the weft branch's history behind step 4's commit".
   - Summary: D10's Rationale was left as it was.
   - Evidence: the Rationale ("the stopped run's records must not stay there") still agrees with "no member's weft inherits" in the checkout sense. But it is also the twin of the new history-reachability claim, and that claim depends on a member's weft being forked from the beam's weft branch tip, which includes that history.
   - Evidence: D5 says the weft branch forks "from the parent's weft branch", so a member's weft history contains the run's records too. That makes "reachable only in the weft branch's history" true of every member's weft branch as well, not only the beam's. A reader may take "no member's weft inherits its … records" as covering history, and it does not.

Tool calls:
- grep of `_lyx/discussion/decision-record.md` for "ready notice", "nearest live", "force", "step 4", "warp commits" and "Info": my context held only the line ranges the fixer read and edited, not every sibling mention of the changed concepts.
