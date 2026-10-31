```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. Find pairs of decisions in the sampler discussion record that act on the same state, and check whether both still hold.

**Pairs checked:**
- D1 × D6 (Finalize's reroute target and D6's raised entry stamp): broken. F1.
- D6 stamp × D6 `goto` (leaving the row without a Done and coming back): broken. F2.
- D4 × D6 bound (the admitted strand and a session that `lym reed resume` reopens): broken. F3.
- D3 × D1 (the perch-off readers named in D3 and the rows D1 declares): broken. F4.
- D7 × the Constraints watchdog fact × the Open risks second bullet: broken. F5.
- D5 × D6 (PR-Review's publish guard reused on a row that neither gates nor pushes): broken. F6.
- D1 × D3 (perch `entry_gate` against Finalize's `on_reroute`, `code_review` false): held. `rerouteReviewed` false makes Finalize halt.
- D1 × D3 (`code_review` true, budget above 0, reroute to Code-Bouncer): held. `enter` evaluates the gate again because `EnteredRow` differs.
- D1 × D2/D3 (Describe has no gate, under a Code-Review budget of 0): held.
- D2 × D3 (ladder profile blocks, no `defaults` entry for sampler, `code_review` default false) and the D2/D3 acceptance bullets: held. The values agree everywhere.
- D4 × D5 (the row records the strand again on its own resume, D4 reads the current one): held for resumes the row performs.
- D4 × D6 (`decision add` while halted at the row): held. The current row is still Sampler-Session.
- D5 × D6 pause (a covered wake stops the session, the uncovered wake runs `ensureSession` and records it again): held.
- D5 × D6 dead strand (Stuck, then `lym loom resume` re-enters and D5 resumes the recorded session id): held.
- D6 × D7 (the watchdog exemption and the row's own dead-strand halt): held.
- D6 stamp × halt and resume: held. The Done count does not change.
- D6 dirty check × D4 `_lym` commits × D8 commits: held. The check covers the code checkout only.
- D6 record check × D4 appended entries: held. `decision add` already runs the same discussion check.
- D6 wiring × D1 `verify_source` × D3 perch: held. Tree scope and attempts come from `darn.yaml`, as D1 states.
- D8 × D4, D8 × D6, D8 × D9 (what the stencil tells the agent against each mechanism): held.
- D1 × D4 (`--amends-plan` refused by rows): held. D2 × the board (refused on a project): held.

**Findings:**

**F1** — MEDIUM, consistency. Location: D6 first bullet, and the D6 acceptance bullet "a done flag written before a reroute or `goto` back to the row".
D6 says Finalize's reroute returns to the row and raises the stamp.
D1 keeps `on_reroute: Code-Bouncer`, which comes after Sampler-Session on the path, so no reroute ever re-enters Sampler-Session.
The reroute half of the bullet and of the acceptance criterion describes a path that does not exist.
Evidence: decision-record D1 line "`Finalize` keeps `on_reroute: Code-Bouncer`"; `contracts/recipes/darn-recipe.yaml` (`on_reroute: Code-Bouncer`).

**F2** — MEDIUM, design. Location: D6 entry-stamp rule.
The stamp is the number of Done history entries for the row.
Suppose the operator or agent runs `sampler-done` and a `goto` moves the run off the row before a wake reads the flag (for example, while it is halted at the row).
The row has no new Done entry, so a later return to the row leaves the stamp unchanged, and the stale flag ends the row at once.
That contradicts the acceptance bullet saying a flag written before a `goto` back to the row is ignored.
The record names only "a return to the row after it ended Done" as raising the stamp.
Evidence: `internal/shedengine/run.go` `appendOrFold` and `foldsInto` (Done entries are counted, not folded); decision-record D6 stamp bullet against the D6 acceptance bullet.

**F3** — MEDIUM, design. Location: D4 "The row re-records the strand on every resume or restart of the session, so the admitted strand is always the live one", against D6's bound "the stop chain or `lym reed resume` may reopen the session".
A session that `lym reed resume` or the stop chain reopens while the run is halted at the row is not resumed by the row, so the row records nothing new.
If the reopened session runs under a new strand, D4 refuses that live agent's `--by operator`.
If it keeps the old strand name, D4 holds.
The record states neither case, so "always the live one" is not established.
Evidence: decision-record D4 and the D6 Bound paragraph; `internal/landingshed/prsession.go` `ensureSession` (only the row's own start records the strand).

**F4** — LOW, consistency. Location: D3, "and the landing's `CodeReviewOff` in `internal/loomcli/landingdeps.go`".
`CodeReviewOff` is read only by Parent-Align, and D1 gives sampler no Parent-Align row.
Naming it as a reading that applies to a sampler run is a dead reference.
Evidence: `internal/landingshed/parentalign.go:435`; `internal/landingshed/deps.go:320`; decision-record D1 "no Parent-Align row".

**F5** — MEDIUM, consistency. Location: the last Constraints fact, against the second Open risks bullet.
The Constraints state as a code fact that a row waiting in process with no exemption is killed after `step_idle_timeout_min`, and that D7 exempts Beam-Await and PR-Review too.
The Open risk then doubts whether those two rows were already protected some other way and leaves the question unchecked.
The code read answers it, so the risk line contradicts the record's own Constraint.
Nothing in `agentActivityFor` names either row:
- a PR-Review session started by `StartSession` writes no wait marker;
- its activity counts only through transcript and events file times while the starting step's process lives;
- Beam-Await logs only on failures, so its trace file mostly stays quiet;
- `reviewWaitingFor` only feeds the status display.

Evidence: `internal/loomcli/looparm.go:125-190`; `internal/shuttleengine/activity.go:115-126,149-178`; `internal/shuttleengine/waitmark.go:239`; `internal/loomcli/reviewstatus.go:163`; `internal/pawlengine/beamawait.go` (log lines are Warn only).

**F6** — LOW, design. Location: D5, "it gets the publish shell guard PR-Review's session gets", against D6, where the row neither gates nor pushes.
The guard's steer text tells the agent "lym pushes the task branch after it gates your commits, and only lym opens, merges or closes the pull request".
In a sampler session nothing gates or pushes at that row, so the reused guard tells the agent something false.
D5 does not say whether the steer text changes for this row.
Evidence: `internal/shellguard/rows.go:100,246-253`.

**Tool calls:**
1. `grep` for publish rows in `internal/shellguard/*.go`: my context did not have the content and steer of the publish guard D5 reuses, which I needed for the D5 × D6 pair.
