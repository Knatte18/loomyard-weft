# raddle splits a Sketch from its stored record and refuses a split that follows the body

`raddle-stale-sketch-record` · Note · bug, shed, loom, board

Imported from [issue #116](https://github.com/Knatte18/loomyard/issues/116).

Seen: a Sketch whose body was rewritten after its motif run (the body folds in later reviews and operator decisions, and supersedes the stored record) was promoted to raddle. raddle built its proposal from the stored decision record, which the run copies to _lym/discussion/decision-record.md, and so split by the superseded design. Told to follow the body, raddle rewrote the proposal from it, but raddle-propose validates criteria, risks, PATTERN changes and decision ids against decision-record.md and refuses it (criterion-uncarried/unknown, risk-uncarried/unknown, pattern-uncarried, criteria-miss-decision). raddle may write only proposal.yaml, and no verb replaces the record: 'lym loom decision add' only appends, and 'decision store' copies the run's record the other way. Related: #114 (no verb replaces a Sketch's record).

Workaround: none inside lym; the run is held while the operator chooses between re-running a motif from the body or replacing the record file in the run worktree by hand.

Expected: a verb that replaces a Sketch's stored record (from a file, by the operator or parent orch), which raddle then reads; or raddle validating against the body when the entry's body is newer than its record.