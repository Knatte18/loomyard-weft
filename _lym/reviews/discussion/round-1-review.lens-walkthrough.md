```yaml
lens: walkthrough
outcome: reported
```

I walked each decision's mechanism through one concrete case. Four decisions meet their purpose. D4 breaks on two counts: a driver that rebuilds the stop from the status file, and a pawl run.

**List: each decision, the case used, purpose met or not**
- **D1** (case: a loom Finalize whose merge of the moved parent conflicts). Purpose met.
  `NewFinalize` builds the resolver for every recipe (finalize.go:160), so the conflict session runs under the lock. `conflicted` is set and the merge record is written. With the switch gone, the run goes on to the gate. A failing gate runs the finalize-mode fixer. A spent cap reroutes through `RerouteReviewed`, or halts for pawl.
- **D1, the retry** (case: `ErrMergeInRequired` with a code-side commit on the parent). Purpose met.
  `mergeAndGate` re-merges and gates (finalize.go:322-325), and with the reroute switch removed it lands.
- **D2** (case: a darn run whose `ReviewInput` falls back to the shared seam). Purpose met.
  `fixerReportInput` stats `Parent-Align-fixer-report.md` and the retired name, and answers not found while absent (subjects.go:234-244, 256-258). That matches the record's claim.
- **D3** (case: a fixer `escalate` at loom Parent-Align). Purpose met.
  The new prefix is not matched by `landingshed.HandsBack` (handback.go:16-23), so `arbiterstop.HandsBack` is false. The loop's envelope `hand_back` (arm.go:309) and the status-built `Stop.HandBack` (driverules.go:115) both stay false. Batten's seam (wire.go:1066-1068) gets the extra predicate. `Eligibility` admits a halted blocked stop that is no landing halt.
- **D4** (case: the same escalate stop reaching the arbiter-first row). Purpose met only for a stop built from a step envelope. It is missed for a stop built from a status (finding 1).
- **D4** (case: a pawl Finalize fixer escalate settled by the arbiter). Purpose not met by any mechanism (finding 2).
- **D5**: docs only, nothing to execute.

**Findings**

1. **MEDIUM, `design`, D4.**
   "The reason being the stop's reason, which the driver's stop already carries" holds only for a stop decoded from a step envelope. A stop built from a status carries the halt text in `Error`, and its `Reason` is empty unless the run is paused.
   - Evidence: driverules.go:86 sets `Reason: decoded.Reason`. driverules.go:110-116, `stopFromStatus`, sets `Reason` to `""` for a blocked status and keeps `Error: st.Error`.
   - The driver builds such stops at the start check (drive.go:629), on a moved re-check (drive.go:776, 781) and at start from status (drive.go:871).
   - Walkthrough: a driver restarted, or newly started, at a halted escalate stop calls `JudgesFirst(producer, stop.Reason)` with `""`. The check is false, so the stop meets the tackler first. That breaks the Webster-like arbiter-first route the acceptance criterion requires.
   - Other rows that read a reason already check both fields: `budgetExhausted` at driverules.go:254, and the status-built `HandBack` reads `st.Error`.
   - The record must say that the reason passed is `Reason`, or `Error` for a stop built from a status. An acceptance criterion must cover a driver that starts at the halted stop.

2. **MEDIUM, `design`, D4.**
   "A pawl run has no decision record to add to, so its arbiter hands an escalate on" is stated as fact, but no mechanism makes it happen.
   - Evidence: the pawl arbiter's recipe text advertises `lym loom decision add` as a repair verb (arbiterprompt.go:11-14, 22). On pawl the verb is refused with "the run has no decision record" (decision.go:653-654). Pawl's fixer is handed `loomengine.DiscussionDecisionRecord(l)` (landingdeps.go:245), a record no pawl row writes.
   - Walkthrough: the arbiter may instead `resume` at the halted row with no decision. `judge` gives the escalate turn back (fixerrun.go:256-257, `s.rec.Spent--`). The fixer re-escalates under a new stop key, and the loop ends only at the arbiter's spawn cap, not at a hand-on.
   - The record should pick one fix:
     - state the bound as the spawn cap;
     - keep pawl's escalate out of `JudgesFirst`;
     - change the pawl recipe text.

Tool calls:
- driverules.go and drive.go, for how a step-envelope stop fills `Reason` against a status-built one. The context showed only `stopFromStatus`.
- subjects.go around `fixerReportInput` and `reviewInputFile`, for D2's not-found claim. The context had only the `overlapNoteInput` grep line.
- (The third was the same subjects.go read's grep of the `ReviewInput` wiring sites, run in that same call.)
