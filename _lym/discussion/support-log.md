# Support log — dobby-run-watch

## Interview

Autonomous mode: no operator turns. Every question below was settled by the writer.

### Turn 1 — what already notifies, and what is missing

Exploration found five existing senders: batten's state-changed and driver-dead notices for a halted child, the driver's hand-back lines, the dobby's run notices for a running child (driver dead, API error, usage limit, quiet), `notify_parent_on`'s row lines, and the opt-in landing notice.
The entry's #207 example was fixed by `77630a4f1` (judged stops now notify); the watch does not depend on it.
Real gaps: a batten halt outside Run-Shed derives as running or landing and sends nothing; the child's done and a landing send nothing unless `land_notice` is on; and no process compares a slot's age with its tier timeout, while the quiet notice stays silent during a live verify wait.

### Turn 2 — shape of the watch

Picked a reading of derived state, batten state and row, child state and row, compared per tick against a memory in the dobby's state record, with a per-task level setting.
Picked additive lines over replacing batten's notice, since batten's holds carry the detail the orch acts on.

### Turn 3 — gate-hold watch

Picked the dobby's probe tick, holder-to-run matching by worktree path, a factor on the tier timeout with a fixed fallback, and awake time.
`dotnet-gate-fixes` (T4) owns the tier timeout's tree kill; the watch reads only slot records and `/proc`, so the two share no code.

### Turn 4 — lens review

Six forks returned reports; none was missing.

- interactions 1, existing-paths 1, walkthrough 1, bound-breaker 4 (done-run clear drops an owed landed line): fixed in D1 (watch before the clear) and D4 (the clear keeps a run owing a line).
- interactions 2, bound-breaker 3 (gate-hold notice shares the run-notice topic): fixed in D6 (own topic).
- interactions 3 (batten row moves after the child's done post under `stops`): fixed in D2 (a row move alone posts nothing under `stops`).
- interactions 4 (an owed line skipped when the reading moves on): fixed in D4 and D5 (owed lines kept in order).
- interactions 5 (memory for the `land_notice` skip): fixed in D4 (a skipped reading still becomes the memory).
- interactions 6 (gate-hold dedup memory home): fixed in D6 (kept in the run's state entry).
- interactions 7 (`off` against the gate-hold notice): fixed in D2 and an acceptance bullet.
- existing-paths 2, walkthrough 5, bound-breaker 5, leanness D2 note (bad-value handling contradicts itself and the existing check): fixed in D2 (joins `beamsetting` and the per-task check; a failed read keeps the last level).
- existing-paths 3 (`run_watch` category): fixed in D2 (`CategoryNone`).
- existing-paths 4 (skip reads `land_notice` afresh): fixed in D3 (reads the landing notice's memory).
- existing-paths 5, bound-breaker 8 (prefix match on sibling worktrees): fixed in D6 (whole path segments).
- walkthrough 2 (absent child after teardown posts a spurious line): fixed in D1 (absent child keeps its last part).
- walkthrough 3 (no pair-dir source on quiet ticks): fixed in D6 (resolved from the slug as the file watch does).
- walkthrough 4 (orphaned testhost outside the holder's tree): fixed in D6 (tier run records its marker; notice names marked processes).
- walkthrough 6, bound-breaker 1 (second stop at the same row missed): fixed in D1 (episode stamp).
- bound-breaker 2 (lines dropped within seconds while the orch is down): fixed in D5 (wait without attempts while no strand; probe spacing; one-hour cap).
- bound-breaker 6 (unrecorded suspends): fixed as a stated bound in D6.
- bound-breaker 7 (`Holders` lock probe): fixed as a stated bound in D6.
- bound-breaker 9 (upgrade burst): fixed as a stated effect in D4; the hibernate part rejected: a pause derives from the batten's pause intent or a shed pause, and one line per paused run is the entry's ask.
- board-scope 1 (key-name helper of `config-keys-nested`): fixed as a plan-writer note.
- board-scope 2 (beam member routing, T12): fixed as an Open risk.
- leanness, notes bullet hiding a rule: fixed, moved into D6 with an acceptance bullet.
- leanness, #207 constraint: fixed, cut from the record; kept here in Turn 1.
- leanness, Scope Out restating D8: fixed, now points at D8.
- leanness, constraints restating decisions: fixed, run-fsm, config-layers and child-processes bullets cut.
- leanness, T4 open risk: fixed, moved to Turn 3.
- leanness, double-mail open risk: fixed, shortened to mail volume.
- leanness, Goal bullets: rejected; the Goal states the entry's own points, which the board-scope lens checks against.
- leanness, Auto-mode values: fixed, each now points at its decision.

## Rejected alternatives

- Replace batten's state-changed notice by the watch: loses batten's stop-report and arbiter holds, and touches the autonomy flow.
- Post the watch line only when no other notice of the stop went within a grace: needs a shared ledger across batten, the driver and the dobby, and arbiter holds can be long.
- Tag watch lines with the run's notice topic: a newer line would replace an unread one, and teardown would drop the landed line.
- A new loom key for progress, or `"*"` in `notify_parent_on`: the row line asks the parent to brief the row's agent, which progress does not want, and loom keys are being nested by `config-keys-nested`.
- Kill a hung gate from the dobby: the tier timeout owns that (`dotnet-gate-fixes`), and a watcher that kills widens its reach.
- Wall-clock age for a slot: a suspend inflates it while the tier timeout's own clock stands still.
- Default `run_watch: off`: the entry asks lym to watch every run unasked.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Does #207 still stand? Resolved by research: fixed in `77630a4f1`. Recorded in Turn 1.
- Q2 Watch level default (auto-pick): `stops`, because the entry wants every run watched without setup.
- Q3 Duplicate mail with batten's notice (auto-pick): accept one extra short line per stop; the orch role keeps it to one fork.
- Q4 Line prefix (auto-pick): `[run watch]`, distinct from `[dobby notice]`, so the orch tells a state line from a running-child condition.
- Q5 Landed line versus `land_notice` (auto-pick): the watch skips landed and cancelled when the landing notice's memory holds the run as requested.
- Q6 Gate bound (auto-pick): factor 1.5 on the tier's `timeout_min`, else 90 minutes; factor 0 turns it off.
- Q7 Clock for the slot's age (research): awake time, since Go timers on Linux run on `CLOCK_MONOTONIC`, which excludes suspend per clock_gettime(2).
- Q8 Where the gate keys live (auto-pick): hub keys of `beam.yaml`, beside the dobby's other keys, not per task.
- Q9 Naming orphaned gate processes (research then auto-pick): a tier run marks every process through an environment entry (`internal/gatecli/processcap.go`); recording it in the slot holder lets the notice name orphans through `proc.PIDsWithEnv`.
- Q10 Retry while the orch is away (auto-pick): wait without counting attempts while no strand is recorded, capped at an hour of awake time.
