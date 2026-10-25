```yaml
lens: fix-consistency
outcome: reported
```

Task: check that round 2's changes to the decision record and support log agree with the sections they touch (fix-consistency lens).

Findings:

1. **MEDIUM.** Changed text: D4, "for a run its state record names owned elsewhere, only the full pass at the lift of a covering pause", and the matching acceptance criterion "drives the run again only after a full pass at the owner pause's lift".
   The lift pass is called a full pass, but D4's own definitions make it a continue pass, so it would skip the reconcile that judges the owner.
   Evidence:
   - D4 defines a continue pass as "A pass dobby schedules into a running batten run at Run-Shed".
   - D4 lists the full passes as "the first pass of each run after a dobby starts, the first pass of each run after dobby sees a machine wake, and every hand pass". A lift pass is in neither list.
   - The new owned-elsewhere paragraph clears the owed full pass, and the `--continue-pass` bound admits a continue pass whenever no full pass is owed.
   - The new `PATTERN-owner-machine` change repeats the same list of resumes, so it leaves the lift pass out too.
   - The list of full passes, the lift trigger bullet ("when a tender pause entry covering the run is lifted") and the PATTERN change text all need the lift of an owner-elsewhere run added. Alternatively, the owned-elsewhere handling should leave a full pass owed instead of clearing it.

2. **LOW.** Changed text: D4's owned-elsewhere paragraph ("runs no further pass of that run until…").
   Two sibling sentences were not updated and still give such a run a pass or a full pass.
   Evidence:
   - "A dobby start owes every run a full pass the same way" and the trigger "at once when the dobby starts" still apply to such a run.
   - Dobby's wake handling "owes every run it drives a full pass" also still applies.
   - Neither sentence excepts a run the state record names owned elsewhere. Whether a restart or a wake re-runs it, and repeats the pause and owner handling, is left unsaid.

3. **LOW.** Changed text: the same paragraph's "runs no further pass of that run".
   D13's definition of in flight was not updated for a run dobby no longer passes.
   Evidence:
   - D13 reads "In flight here means a run dobby passes, `running` and with no cancel marker". A `running` run held owned elsewhere fits the second half but is no longer passed.
   - D5's `max_runs` count ("runs it drives are in flight, halted ones included") has the same gap.
   - The record does not say whether such a run holds off the backoff or takes a `max_runs` slot.

4. **LOW.** Changed text: D4's F2 sentence (an owned-here verdict clears `ownedBy` and the pass spawns), next to the F1 paragraph.
   A takeover now has two routes that dobby handles differently, and the record never contrasts them.
   Evidence:
   - On the first route, InnerRun's `ownedElsewhere` runs inside a continue pass. Dobby goes on passing the run, which yields at the hold, as the new criterion "held at the spawn on a standing owner and then adopted back is respawned by the next pass" requires.
   - On the second route, a full pass's refusal makes dobby stop passing the run.
   - Nothing says the first route leaves dobby's state record without the owned-elsewhere mark, so a reader can apply F1's "no further pass" to it.

5. **LOW.** Changed text: D2's new `lyx adopt` refusal.
   Two siblings were not updated for this new refusal in CLI behaviour.
   Evidence:
   - Scope's In list names `lyx adopt` only as a caller of `lyx beam start`, not as gaining a refusal.
   - The support log's Question ledger has no entry for this pick, or for F3's pick of the PATTERN change over dropping the skip. Round 2's support-log entries do state the F7 rationale.

No further tool calls were made.
