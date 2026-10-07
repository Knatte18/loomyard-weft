---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:82-83 and :153
    summary: D5 and the Auto-mode assumption name the receiving set by activity (loom, webster and burler wiring) although webstercli and burlercli each build their own runner, so an autonomous recovery strand spawned by lyx webster recover-batch is in neither the notifying set nor the wire-none list
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:82
    summary: D5 names batten-driven runs as reaching the notifier although battencli and battenshed build no runner; the clause is redundant with the runner-based statement and names a site that has no runner
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:14, :69 and :174-177
    summary: The Scope bullet and the D5 title state the cadence as once per held turn end, while D5 body makes an Attach notify only the last held turn end past the notified offset, and no acceptance criterion pins that exception
---

### [MEDIUM] Receiving set is stated by activity, not by runner, and misses two runner sites

**Location:** `_lyx/discussion/decision-record.md` D5 (lines 82–83) and Auto-mode assumptions (line 153).
**Issue:**
The notifier is a seam on `shuttleengine.Runner`, so the receiving set is exactly the runners whose construction site sets it.
`internal/loomcli/wiring.go:425` builds one runner and shares it with `burlerengine.New`, the `runnerMasterStarter`, `reflectionShuttle`, the landing `mergeresolve` `Shuttle` and `runnerDriverStarter`.
D5 and the assumption describe that as "the runner that the loom, webster and burler wiring builds", which reads as three wiring sites and fits no runner boundary.
Two other sites build their own runner and are in neither D5's notifying set nor its "wire none" list (`lyx shuttle run` and `orch start`):

- `internal/webstercli/wiring.go:191` builds a runner for `lyx webster`; `recover-batch` spawns an autonomous recovery strand through it (`internal/webstercli/recoverbatch.go:158`, `c.starter = runner`).
  That process is the Master's own subprocess, so the loom runner never sees that strand, and its asking stop would hold silently to its deadline, the hazard Open risks names for a runner with no notifier.
- `internal/burlercli/wiring.go:135` builds a runner for the operator's `lyx burler`.

The two assumption sentences also say the same thing in different words, with the "wire none" half naming two verbs (`lyx shuttle run`, `orch start`) where `orchcli` builds its runner for every orch verb (`internal/orchcli/cli.go:136`).
**Fix:**
State the set once, by constructing package, in D5, and have the Auto-mode assumption refer to it with the same wording.
Decide the two missing sites: wire the hub-mode runners of `webstercli` and `burlercli` to `orchengine.QueueNotice` on the prime's orch like `loomcli`'s, so the recovery strand notifies; `shuttlecli`, `orchcli` and every detached (standalone) runner wire none.

### [LOW] D5 names batten-driven runs, a site that builds no runner

**Location:** `_lyx/discussion/decision-record.md` D5 (line 82).
**Issue:**
"batten-driven runs reach it through those verbs" names a wiring site that builds no runner.
`internal/battencli` and `internal/battenshed` contain no `NewRunner`/`NewDetachedRunner`; batten drives a child loom driver, whose own `loomcli` runner is the receiver already covered by the runner-based statement.
Naming batten invites a plan writer to look for a seam there.
**Fix:**
Delete the clause; the batten `QueueNotice` wiring stays cited only as the pattern to copy, which the Notes for the plan writer already do.

### [LOW] Cadence headline omits the Attach exception, and no criterion pins it

**Location:** `_lyx/discussion/decision-record.md` Scope (line 14), D5 title (line 69), Acceptance criteria (lines 174–177).
**Issue:**
Scope says the parent is notified once per held turn end and D5 is titled the same.
D5's body then says an Attach notifies at most one turn end from the events already in the file, the last held one past the notified offset, so earlier unnotified held turn ends are never notified.
The acceptance criteria cover "a second held turn end notifies again" and "replaying events after Attach does not re-notify", but not that an Attach over several already-present held turn ends notifies one.
The legacy criterion states the same rule for `asking` records only.
**Fix:**
Qualify the Scope bullet with "live" turn ends and "at most the last one already in the file on Attach", and add an acceptance criterion for an Attach over several held turn ends notifying exactly one.

## Focus departures

None.
Both focus questions were reached: F1 and F2 settle the receiving-set question (D5's wiring sentence and the Auto-mode assumption agree in substance but not by runner, and one names batten), and F3 settles the cadence question (Scope, D5 and the acceptance criteria agree on per-held-turn-end, with the Attach exception stated only in D5).
