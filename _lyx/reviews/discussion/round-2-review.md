---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: A watchdog quit of a healthy package reaches the loom and landing verify gates too, but only webster's refusal row and stuck reason gain the busy_quit_sec way forward
    fix: In D6, extend the watchdog-quit wording beyond webster's gate to every halt that reports a failed verify command run through lyx gate test, namely the loom verify gate (internal/loomshed, refusal-spec rows Webster-Burler-Fix verify gate exhausted and Darn-Write verify gate exhausted) and the landing verify gate (internal/landingshed); each of those halt reasons, when the log carries D7's watchdog line, names the window and raising busy_quit_sec through lyx config gate from the prime, and each matching refusal-spec row gains that way forward in the same commit; add Scope In lines for loomshed and landingshed and an acceptance criterion that a watchdog line in the log of each of those gates' failures yields a halt reason naming busy_quit_sec.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7
    summary: The enclosing-gate-test variable is trusted unchecked, so a detached descendant such as a pane in a test's tmux server, or any process that sets it, runs with no watchdog while no outer watchdog sees its tree
    fix: In D7, make the variable carry the enclosing lyx gate test's pid, and have a lyx gate test skip its own watchdog only when that pid is a live ancestor of its own process, walked through the parent pids internal/proc already reads; state in D7's Bound that a variable naming no live ancestor is ignored, and add to the watchdog acceptance criterion that a set variable naming a non-ancestor pid still runs the watchdog.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: The integration criterion drives an inner lyx gate test from a test that itself runs under lyx gate test, so the inner run inherits the enclosing variable, skips its watchdog, and is quit only by the outer run at the hub's 120 seconds
    fix: In the acceptance criterion that starts Under lyx gate test on Linux, state that the test removes D7's enclosing-gate-test variable from the inner lyx gate test's environment and asserts the watchdog line carries the inner run's 1-second window, so the outer run's watchdog cannot satisfy it.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 and Acceptance criteria
    summary: The integration criterion spins one goroutine, the exact boundary D7's 90 percent threshold is tuned for, so under an oversubscribed machine the fixture can sample under the threshold and the test passes or fails by load
    fix: Move the single-spinner boundary to the injected-sampler criterion, adding that a .test process sampled at 92 percent of one core for the whole window is quit; in the integration acceptance criterion, replace spins one goroutine with spins several goroutines, at least two, so the binary's samples sit well above one core whatever else runs; keep D7's threshold, and add to D7's Rationale that the 10 percent margin covers tick granularity only and a spinning goroutine starved below it by CPU contention falls to go test's -timeout, the case Open risks already names for a livelock below 90 percent.
---

### [BLOCKING] The busy_quit_sec way forward reaches only webster's gate

**Location:** `_lyx/discussion/decision-record.md` D6

**Issue:** D7's watchdog fires inside any `lyx gate test`, so it quits a healthy CPU-bound package in every verify gate whose verify command runs through `lyx gate test`, not only webster's.
`verifytree.Verify` is called from `internal/loomshed/verifygate.go` and `internal/landingshed/verifygate.go` as well as `internal/websterengine/verifygate.go`.
The loom gate's refusal rows (`Webster-Burler-Fix verify gate exhausted`, `Darn-Write verify gate exhausted`, `contracts/specs/refusal-spec.md:330-331`) give "fix the failing tests" as the way forward, and the landing gate's halt reason renders `g.failedWayForward` (`internal/landingshed/verifygate.go:231`).
For a watchdog quit of a healthy package, no code fix applies, so those halts name a way forward that does not fit the case.
D6 changes only the `verify gate exhausted` row.
`PATTERN-refusal-way-forward` requires a refusal to name a way forward that fits, so this is BLOCKING.
For webster's own gate, the focus question is settled: D6's stuck reason, report and findings name raising `busy_quit_sec` through `lyx config gate` from the prime, matching the precedent at `refusal-spec.md:161`, and an acceptance criterion pins it.

### [MEDIUM] The nested-run skip trusts its variable unchecked

**Location:** `_lyx/discussion/decision-record.md` D7

**Issue:** D7 skips the watchdog whenever its new variable is set, "since it runs under an enclosing `lyx gate test` whose tree already holds its test binaries".
That premise fails for a descendant that leaves the outer run's process tree.
`tmuxkit`'s `serverEnviron` (`internal/testkit/tmuxkit/tmuxkit.go:381`) keeps every variable except Claude's, the trace id and the agent names, so a tmux server a test starts carries the variable into its panes, and that server daemonizes out of the outer run's child tree.
A `lyx gate test` in such a pane runs no watchdog, and the outer watchdog cannot see it.
The same holds for any process that sets the variable by hand.
Under the rubric's attack-surface question, this is a guard made bypassable with no stated bound.
The slot inheritance beside it is validated (`Pool.Inherited` checks the lock path), but this variable is not.
On the first focus question: D7 now keys the skip on its own variable, not on the inherited slot, so a `lyx gate test` run as `verifytree.Verify`'s verify command does run its watchdog, and an acceptance criterion pins that.

### [MEDIUM] The integration criterion is satisfiable by the outer run's watchdog

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** Every tagged test runs under `lyx gate test --tags integration`, whose go test environment carries D7's variable.
A test that spawns an inner `lyx gate test` over a spinning fixture passes that variable on, so by D7 the inner run skips its watchdog.
The outer run still watches the fixture as a descendant and quits it after the hub's `busy_quit_sec`, by default 120 seconds.
The output then holds goroutine stacks, a watchdog line and a failing exit, all "well before go test's timeout".
So the criterion passes, two minutes late, without exercising the inner run's 1-second budget.

### [MEDIUM] The integration criterion sits on the threshold boundary

**Location:** `_lyx/discussion/decision-record.md` D7 and Acceptance criteria

**Issue:** D7 sets the threshold at 90 percent of one core on the premise that a spinning goroutine's samples fall "just under one core" from tick granularity and slot neighbours.
Tick granularity costs about 1 percent over a 1-second sample at 100 Hz, but CPU contention has no such cap.
The hub template runs 2 slots × `-p=4` test binaries, each with `GOMAXPROCS` at the machine's core count and its own `t.Parallel` tests, beside other hubs' gates.
On an oversubscribed machine, CFS gives the single spinning thread less than 90 percent of a core.
In production, that miss only falls back to go test's `-timeout`, which is acceptable and close to the livelock-below-90-percent risk already named.
The integration criterion's single goroutine, though, makes the test's pass depend on machine load, against `PATTERN-test-speed`'s determinism.
Round 1's F2 chose one goroutine to pin that a single spinner is caught.
That intent stands, but a sampler a test injects pins it without load dependence, so the boundary moves to the unit criterion rather than being dropped.

## Rejected

- D5 and D6 on a non-signal exit of 128 or above (fourth focus directive): not a defect.
  D5's Bound names git's 128 and a tool's 255 as hangs that skip only the one rerun, and D6's exit-code wording, "killed or exited with that code", stays true for a non-signal exit.
  `lyx gate test`'s own slot-busy exit is 75, below the 128 cut.
