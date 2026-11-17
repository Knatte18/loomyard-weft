```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checking that the changed text still agrees with the sibling cards, decisions and cross-references.

**LOW**: `_lym/plan/00-overview.md`, Shared Decision `nil-seams-keep-today`, Decision line rewritten by F8.
- **Summary:** The Decision line now says each zero value "turns its own part off", and that a nil `BurlerHalfDeps.Snapshot` still runs incremental rounds under the BLOCKING rule alone. The decision's slug `nil-seams-keep-today` was left as it was, and so was its Rationale, which says hand-built receivers "keep reviewing and bouncing as today". Both now claim more than the reworded Decision line.
- **Evidence:** Line 32 reads `### Decision: nil-seams-keep-today`. Line 35 reads "Hand-built receivers in existing tests and an in-flight run that crosses the binary change keep reviewing and bouncing as today until they hold a complete new-format round". Card 4:44 says a nil `Snapshot` still writes the record without a target id. Card 7:54 builds `profile.Round` from that record whether or not a Snapshot is set. So a receiver with a nil Snapshot does not keep reviewing as today once it has a complete round. The "until" clause covers only part of that.

The other changed passages agree with their siblings:
- Card 2's strict top-level keys agree with Card 10's `roundUnchanged`, which counts an unparseable report as changed, and with Card 7's nil-context rows.
- Card 1's seeded index agrees with D5 at decision-record.md:194 and :215.
- Card 11's bound agrees with Card 10's bound and with the `softCapRule` doc.
- Card 6's new sentence agrees with its line 18 and its nil-context test row.
- Card 8's changes and Card 12's overview line add no contradiction.

Tool calls:
- grep of the plan cards and the decision record for the lenient, ignored, cause-count and unreadable claims. My context did not hold the cards I had not read (03, 04, 05, 07) or the record's D5 text.
- grep of cards 03 to 12 for `WorktreeTreeID`, tracked files, `Profile.Round` and today's-text claims, to find twins of the F2 and F8 changes in cards my context did not hold.
