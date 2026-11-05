```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked**
- D2 bound "a member filed in that window never starts, so nothing lands into a ready beam" — **broken** (hand `lym batten run`).
- D2 "dobby's start hold" as the ready-window member guard — **broken** (same).
- D3 bound "after ready only the parent and GitHub can move" — **broken** (same counterexample).
- D3 refusal 6 "a pull request merged on GitHub passes, since Finalize lands it through the already-landed path" — **broken** (merged at an older head).
- D3 refusals 1–5 — held.
- D4 "a record naming another slug reads as no record" — held (as `ReadyFor` filters).
- D6 bound "change nothing once the ready record stands" — held.
- D7 bound "a fresh machine repeats at most one notice" — held (one line, possibly listing already-announced commits).
- D8 "a crash between the GitHub merge and the parent-side landing re-enters through the already-landed path" — held (record dropped only after the landing).
- D10 bound "it adds no way to take new work" — **broken** (hand-run counterexample).
- D10 "the dobby lands the beam again once nothing works under it" — **broken** (the dobby's closed land episode persists).
- D11 "the dobby writes the land request again from the tracked ready record" after a crash between the accept's removals and Done — **broken** (same persisted episode).
- D11 "completed wake resets repair cap" — **unbounded** (no stated bound on repeated re-steps).
- D11 "arbiter's eligibility … extends … to pawl" — held but vacuous: `arbiterstop.Eligibility` has no recipe gate; the policy table alone admits it.
- D12 bound "`lym beam start` never starts the dobby of a held beam" — held.
- D13 bound "only a status naming a row the recipe lists as retired moves" — held.
- D13 "a run at Finalize or past it is untouched" (and "goes on as before") — **broken** (D8 changes how such a run's Finalize decides).
- D13 ready-record removal — held as mechanism; commit not stated (out of this lens).
- Constraint "a push lock whose holder process died is free already" — held (`lock.go` uses gofrs/flock).

**Findings**

1. **BLOCKING · design · D11 (and D10's bound)** — The dobby never rewrites the land request after an accept the run did not finish, so Beam-Await waits forever with no hand step named.
   Counterexample: `landBeam` records the episode as acted (`State.Land{Refused:false}`) when it asks for the landing, and clears it only on a reading that does not pass (`internal/dobbyengine/land.go:41-52`); the record is a state file kept across dobby restarts (`land.go:79-86`, "neither this dobby nor a restarted one acts on it again"). After the accept's removals (or after a goto back to Beam-Await), the ready record still stands and nothing works under the beam, so the reading keeps passing and a restarted dobby returns at `if record.Land != nil && !record.Land.Refused` (`land.go:50-52`). Today's code knows this: the interrupted-landing notice names a hand `lym beam land` (`internal/pawlengine/beamawait.go:194`). D11's claim "the dobby writes the land request again from the tracked ready record" and D10's "the dobby lands the beam again" are both false, and issue #57's no-hand-step purpose is missed.

2. **MEDIUM · design · D2 bound (also D3's and D10's bounds)** — A hand `lym batten run <member>` starts a member under a ready beam and its work lands into the beam.
   Between `lym beam ready` and the accept the status names Beam-Await, which is hold-free (`internal/boardengine/runlock.go:42`); the member's fork precondition refuses only on `RunHolds(beamEntry)` and never reads the ready record (`internal/battencli/beamfork.go:65-70`; no ready read in that file). `PATTERN-dobby-starts-runs` itself admits a hand `lym batten run` pass. So a member filed after ready (or released by a goto that lifted the hold) starts and lands, moving the beam head past the head the pull request was checked at and the description was written for. The record never states this path and its bound says the opposite.

3. **MEDIUM · design · D3 refusal 6 / D8 merged path** — A pull request merged on GitHub at an older head passes `ready`, but Finalize's already-landed path does not carry the later commits.
   Counterexample: orch opens the PR; someone merges it by hand on GitHub; a member then lands into the beam; the orch runs `lym beam ready`. Refusal 6 compares heads only for an open PR and passes a merged one outright. The already-landed path applies only when "the parent already holds the task's work" (`internal/landingshed/doc.go:99`, `finalize.go:178`), and today's PR-Review treats a merge at a head other than the gated one as stuck (`internal/landingshed/prreview.go:753`). The record states no head check for a merged PR and no way forward when they differ.

4. **MEDIUM · design · D11 repair cap** — "A loop exit after a wake that completed counts as a new stop" yields an unbounded re-step loop, and D11 states no bound.
   The stop key is (producer, history length, state) (`internal/sheddriver/driverules.go:136-142`); re-steps of Beam-Await never grow history. With the cap reset after every completed wake, a loop that dies after each first wake (a crash in a later wake's code, an OOM from the dobby's reads) re-steps forever at roughly the await floor's cadence, never reaching the tackler, arbiter or parent (`driverules.go:244`). The rubric's attack-surface rule makes a weakened guard with no stated bound a finding.

5. **MEDIUM · design · D13 "a run at Finalize or past it is untouched", with D8** — A pawl run an old binary left at Finalize after PR-Review opened a pull request does not "go on as before".
   Its PR lives only in the run's scratch record; D4 says the scratch record serves only what Finalize opens itself, and D8 decides a pawl landing from D4's record alone. Under beam.yaml `on`, D8's third bullet squash-merges locally and pushes, leaving the PR that PR-Review opened (possibly under review) open and orphaned and its review bypassed.
   Evidence: D8 bullets (decision-record.md:120-130); PR-Review's scratch record (support-log.md:14); `internal/landingshed/finalize_pr.go:140` (today's Finalize merges the scratch-recorded PR on GitHub).

Tool calls:
1. grep `internal/battencli/beamfork.go` + `internal/pawlengine/ready.go` for `RemoveReady`: whether a hand member run checks the ready record and whether removal commits; the manifest had the fork check but not its ready-record coverage.
2. grep of where the dobby resets `State.Land`, plus `postLandNotice`/`recordLand` in `internal/dobbyengine/land.go`: how the dobby's land episode closes and persists; the manifest held only land.go 1-68.
3. grep for the already-landed path in `internal/landingshed`: needed to attack D3 refusal 6's merged-passes claim; not in the manifest.
