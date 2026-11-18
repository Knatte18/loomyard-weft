```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the F13 change set agrees with its sibling decisions, criteria and cross-references, with the fix-consistency lens.

**[LOW] Question-ledger twin still says the reroute clears the session id**
- **Changed text:** D4 now has the row's entry after a persisted send-back reroute clear the session id, and the D4 criterion at decision-record line 206 was changed to match.
- **Summary:** The support-log's question ledger still says the reroute itself clears it.
- **Evidence:** The ledger entry in `_lym/discussion/support-log.md` reads: "Does the re-entered PR-Review resume the old session? Auto-pick, after round 1: no; the reroute clears the session id, so a new session starts."

**[LOW] D4's Rationale list of new parts leaves out the injected status reader**
- **Changed text:** The fix adds "the landing wiring injects a reader of the run's shed status into the row".
- **Summary:** The fix adds a new seam, but the list of new parts in D4's Rationale was not updated.
- **Evidence:** D4's Rationale says: "New are the mark, the verb, the stored file, the full generation forced by a review input, the seam's list of inputs and the review-input note." The status reader is not in that list.

I checked these and they agree with the change:
- D2's Bound, line 49, says "between send-backs … with the gaps D4's Bound names".
- D5 says "between two send-backs, or before the first".
- Line 125 says "With the session id cleared it starts a new review session".
- The dirty-tree criterion, line 209, and the reroute-cap criterion, line 212.
- The `goto` Bound sentence.
- The support-log Round 3 entry.

No further tool calls made.
