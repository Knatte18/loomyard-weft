```yaml
lens: interactions
outcome: reported
```

**Lens: interactions** — I checked every pair of decisions that act on the same state, plus each acceptance criterion and constraint against the decisions it touches.

Pairs checked:
- D1 join × D3 lease (several fronts writing one lease file): **broken**, the lease is overwritten rather than extended (F1).
- D3 lease × D5 long cap (a 55m-cap front and a 250s-cap front on one run): **broken**, same defect as F1.
- D3 abandonment and D1 supersede × D5 median (killed calls in the run records): **broken**, the records cannot mark a killed call (F2).
- D1 supersede × D3 lease expiry (the front writes into the old run's subdirectory; old and new runners hold slots together): held, and the bound is stated.
- D1 "result delivered" × D3 multiple fronts (two fronts with the same words, e.g. forks sharing a worktree): held for correctness, but costs a full extra run (F3).
- D1 75→1 remap × D1 replay bound × D6 card-verify exit pass-through: **broken**, the bound contradicts the remap (F4).
- D2 strand/cap gate × D4 zero cap and `DefaultAwaitCap` fallback: held. Validation forbids a configured zero, so zero comes only from an engine with no cap, and an unparsable strand gets the 250s fallback, which is non-zero.
- D2 × D7 (agent renders drop `--wait`, Go renders keep it): held. `loomcli/roundgate.go` serves both a Go verify call and a note an agent reads; D7's rule sorts them per site.
- D6 × D7 (a card's `**Verify:**` line keeps `--wait` and runs as written inside the card-verify runner, without the strand): held.
- D6 × D1 join key and tree check: held, D1's tree and dirty-set check applies to card-verify too.
- D6 × D9 (card-verify's new 75 read as "run again"): held.
- D3 lease × D2 slot wait (the runner's slot wait has no cap but is bounded by the lease): held.
- D4 × D9 (cap below the 1h lifetime; long cap 55m &lt; 3600000 ms Bash timeout): held.
- D4 × D5 (break-even = role cap × `keep_warm_max_reruns`; long cap outside the cap validation by design): held.
- D4 `lym shuttle await` default × D8 Master waits (template keeps 250s, so the Master's waits are unchanged): held.
- Constraint gate-slot × D1 (the runner runs the same in-process `runTest` with `--wait`): held.
- Constraint tolerant-records × D5 (records without a call id are left out of the median but still totalled): held.
- Constraint daemon-wakeups × D1/D3 (front poll and lease read at most once a second): held.
- Acceptance D1 "third call starts a fresh run" × D1 delivered rule: held.
- Acceptance D3 "signal to a front leaves the runner going" × D2 (operator calls stay in-process): held.

Findings:

**F1 — MEDIUM, design, D3 (lease) × D1/D5 (fronts with different caps)**
D3 has each front write the lease as "its own cap deadline plus a grace of 120 s". That is an overwrite, not "the later of the current lease and mine".
- Counterexample: front A waits with D5's long cap (55m) and writes a lease of about 57m. Front B joins the same key and writes its own short-cap lease of about 6m. B can have the same words in the same worktree but a different role cap, or a median that crossed the threshold between the two calls.
- Its 250s cap and 120s grace then pass. The runner reads the lease, abandons, kills the tiers and records exit 1 "no caller came back", all while A is still blocked inside its 55m wait.
- A gets a failure verdict that no failing test produced, which breaks D5's bound that a wrong pick "costs cache money only, never a verdict".
- The fix is to state the renewal rule as the later of the two deadlines. D1's supersede path should stay the one write that may shorten the lease.
- Evidence: decision-record.md D3 ¶1 ("Each front writes the lease … its own cap deadline plus a grace"), D3 ¶2 (abandonment exit 1), D5 Bound.

**F2 — MEDIUM, design (deferred fact), D5 × D3/D1**
D5's median leaves out "an abandoned call", and D1 kills a superseded call the same way. Both end the call as a terminating signal does today, and no run-record field marks either case.
- `gateslot.RunRecord` has only `Outcome`, `ExitCode`, `TimedOut`, `Carried`, `Build` and `BlockedBy`; there is no cancelled or abandoned flag.
- The abandonment is recorded only in the runner's exit file under `.lym`, while D5 reads `ReadRunRecords` from the board's gate directory.
- A call killed during its last tier may therefore have a record for every tier it listed, with a short wall time. D5's "ran every tier they listed to an outcome" would count it, which drags the median down.
- The record should say how a killed call is told apart: a new record field, or the call id cross-checked against the exit file. Today this is left to Plan to find out.
- Evidence: internal/gateslot/runlog.go:38-73 (RunRecord fields); decision-record.md D5 ¶2, D3 ¶2, D1 ¶2.

**F3 — LOW, design, D1 "result not yet delivered" × D3 multiple fronts**
D3 expects several fronts to renew one lease, but D1 lets only the first front to read the exit file get the result.
- Two fronts with the same words in one worktree (e.g. two forks of one session) each got a 75. The one that comes back second finds the result delivered and starts a fresh runner, so the whole gate runs again.
- This changes no verdict, but neither Decision states the bound.
- Evidence: decision-record.md D1 ¶2 (join conditions), D3 ¶1 ("Each front writes the lease when it joins").

**F4 — LOW, consistency, D1 Bound × D1 75 remap / D6**
D1's Bound says "The replay gives the same content and exit code as an in-process call". The same Decision remaps a recorded 75 to exit 1 with an extra stderr line.
- For card-verify (D6), the in-process verb today exits with the line's own code, so a line that exits 75 is replayed with a different code and output.
- The Bound should name the 75 exception.
- Evidence: decision-record.md D1 exit-file bullet and Bound; internal/gatecli/doc.go "Card verify" (the verb exits with the line's own code).

Tool calls: none; my inherited context held everything this lens needed.
