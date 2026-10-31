```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix set agrees with sibling text, and find any twin of a change that was left as it was.

**Findings**

1. **LOW.** Changed text: the D6 stamp bullet, now "the number of times the run's status history shows the run entering the row".
   - Twin left stale: `_lym/discussion/support-log.md:115` (Q16) still says "The count of the row's Done entries in the status history".
   - This is the definition that F2 replaced, and the support log is in the fixer's write surface.
2. **LOW.** Changed text: the Constraints watchdog fact, rewritten for F1. It now says the kill comes from `agentActivityFor` having no row exemption, and that a `StartSession` session writes no wait marker.
   - Twin left stale: `support-log.md:39` (Turn 6) still gives the old mechanism: "counts a held turn end as no activity, so a sampler step would be killed".
   - F1 called that mechanism wrong for this session.
3. **LOW.** Changed text: D3, where F8 removed `CodeReviewOff` from the perch-off readings.
   - Twin left stale: `support-log.md:55` still records "Finalize's reroute and `CodeReviewOff` read the budget … fixed, every perch-off reading follows the rows' entry gate too (D3)".
   - D3 no longer names `CodeReviewOff`, and sampler has no Parent-Align row.

The rest of the change set agrees with its siblings:
- The D6 entry-time flag read fits D5. The verb refuses before a session is recorded, so there is no flag on first entry.
- The D6 wake order, the D6 Bound and the acceptance bullets agree. The acceptance line that says "ends the row Done and the session is stopped" fits the stop-first order.
- D1's `on_reroute`, D3's reroute halt and the F6 acceptance bullet agree.
- The F5 change to D4's Bound fits D4's own line that Describe measures against the amended record.

Tool calls:
- grep of `support-log.md` for twins of the changed phrases. My context did not hold the support log.
