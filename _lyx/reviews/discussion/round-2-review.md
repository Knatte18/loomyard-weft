---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D1
    summary: D1 rests on a false premise; every step, run and goto arm rebuilds the shed as loom's, so a darn run armed after bootstrap gets the loom graph and no decision says how an arm picks the darn builder
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D6, D7
    summary: the call-time stop on an empty darn verify is a loom refusal reachable through step, resume and goto, yet the record gives it no way forward and no refusal-spec row, against PATTERN-refusal-way-forward
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4, D7
    summary: verify_attempts below 1 is refused only at lyx loom start, though the darn builder stamps it at every arm; a later set to 0 reaches the gate unrefused, unlike loomrecipe.New, which refuses a budget below 1 on every build
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D10
    summary: the stated bound never on a landed run does not hold on the lyx fabric remove route; a run that merged at Finalize but whose done mark failed is cleared to empty when batten is paused, failed or absent, and the record does not say it lets that through
---

### [BLOCKING] D1: the arm builds loom's shed, and the record decides the recipe is read only at bootstrap

**Location:** `_lyx/discussion/decision-record.md` D1, D4 (budget stamped by the darn builder), acceptance criterion "`lyx shed step` and the four generic verbs arm a `darn` run through `loomcli.ArmAt`".

**Issue:** D1 says the seed's recipe is read in loom's bootstrap, "the one place that already reads the seed to arm", and that every `lyx loom` verb serves a darn run unchanged "because each reads the status file and the records, never a recipe".
The shed is not built only at bootstrap.
`loomcli.ArmAt` and `Arm` fill `spec.BuildShed` per verb: `step` uses `c.buildLoomShed` and `run` calls `loomrecipe.New(c.env, c.shedPaths)` (`internal/loomcli/arm.go:274-278`).
`buildLoomShed` also ends in `loomrecipe.New` (`internal/loomcli/sharedbootstrap.go:288-325`).
Neither takes a recipe.
Mapping `darn` to `loomcli.ArmAt` in `internal/shedcli`'s table therefore arms the loom graph on every `lyx shed step` the driver runs after `lyx loom start`.
That graph has no `Darn` row, so a status whose current producer is `Darn` cannot be stepped.
`lyx loom resume` and `lyx loom goto Darn` hit the same problem.
D4's claim that the darn builder stamps `verify_attempts` "the way the loom builder stamps the review budget" assumes a darn builder runs at arm time, and no decision puts one there.
The arm already reads the seed: `internal/loomcli/arm.go:76-82` describes the `shedrun.ReadSeed` refusal for the generic verbs, and `internal/shedcli/cli.go:255` looks the recipe up from `seed.Recipe`.
So the recipe is within reach at the arm, but the record never decides to use it there.
This is the recipe field, not the driver field, so `PATTERN-driver-choice-single-site` does not stand in the way.

**Fix:** Add a decision covering how an arm picks the shed builder for a darn seed.
Either loom's arm selects the darn or loom builder from the seed's recipe for every verb that builds a shed, or `internal/shedcli`'s `darn` entry passes the recipe in, with `lyx loom`'s own subtree doing the same.
Correct D1's claim that verbs never read a recipe.
Add an acceptance criterion: over a darn seed, `lyx shed step`, `lyx loom resume` and `lyx loom goto Darn` each build the darn shed.

### [BLOCKING] D6: the darn reader's call-time stop is a loom refusal with no way forward and no spec row

**Location:** `_lyx/discussion/decision-record.md` D6 ("returns an error, never `""`, when the file is missing or `verify` is empty, so the darn gate, Publish and Finalize each stop at call time"), D7 ("each a refusal row in the loom section"), Constraints `PATTERN-refusal-way-forward`, acceptance criterion "With `verify` emptied after the start…".

**Issue:** The record names this error as "the guard that holds for every step, resume and goto".
That makes it a refusal reachable through loom's verbs.
`pattern/PATTERN-refusal-way-forward.md` requires every such refusal to name its way forward in its message and to have a row in loom's section of `refusal-spec.md`.
D7 lands rows only for the two `lyx loom start` refusals, and the Constraints entry says "the two `darn.yaml` refusals land with their rows".
D6 does not say what the error tells the operator.
Today the landing gate wraps a read error as `landingshed: <producer>: read verify command: …` (`internal/landingshed/verifygate.go`, `check`), which halts the run failed with no way forward.
The acceptance criterion pins that each site stops, but not what the stop says.
The record's own constraint list misses this refusal, so this is a listed-constraint violation, and the instructions make it BLOCKING.

**Fix:** In D6, give the reader's error a way forward: set `verify` with `lyx config darn --set verify=<command>` from the prime, then `lyx loom resume`.
List it as a third refusal row in D7 and the Constraints entry.
Extend the acceptance criterion so it asserts the message carries that way forward at the gate, Publish and Finalize.

### [MEDIUM] D4/D7: `verify_attempts` is checked only at start, though it is stamped at every build

**Location:** `_lyx/discussion/decision-record.md` D4 ("stamped onto the row by the darn recipe builder, the way the loom builder stamps the review budget"), D7 ("refuses when … `verify_attempts` is below 1"; "the call-time error in D6 is the guard that holds for every step, resume and goto").

**Issue:** The shed is rebuilt at each arm (see F1), so each build reads `verify_attempts`.
D7's start refusal covers only the first start.
A later `lyx config darn --set verify_attempts=0` reaches the `Darn` row's gate at the next step, and D6's call-time guard covers only `verify`.
The precedent D4 names guards the build itself: `loomrecipe.New` refuses a `ReviewMaxBounces` below 1 "as its first act" on every build, naming the config key (`internal/loomrecipe/loomrecipe.go:35-36, 68, 134`).
The record does not say what a gate with zero attempts lets through, and no acceptance criterion covers a change after the start.

**Fix:** Decide that the darn builder refuses a `verify_attempts` below 1 on every build, naming `lyx config darn --set verify_attempts=<n>`, and give that refusal its spec row (it is reachable through step, resume and goto).
Add an acceptance criterion for a value lowered after the start.

### [MEDIUM] D10: "never on a landed run" does not hold on the `lyx fabric remove` route

**Location:** `_lyx/discussion/decision-record.md` D10 ("What bounds it: … never on a landed run").

**Issue:** On the hand-removal route, D10 detects a landing only through a batten run in state `done`.
`Finalize.markTaskDone` only logs a failed done mark and keeps the verdict (`internal/landingshed/finalize.go:246-256`).
A run can therefore merge at Finalize, leave its entry in the run form, and then halt at `Friction-Reflect`.
If the operator removes that pair with `lyx fabric remove` while its batten run is `paused` or `failed`, or when the slug has no batten seed (a hand-started loom), D10 clears the entry to empty instead of marking it done.
The board then shows not-done work that is on the parent branch.
The record's attack-surface paragraph claims the opposite.
D10 also never says that batten stays `running` while its child halts.
`Run-Shed`'s halted arm returns a budget-exempt Stuck that self-routes (`internal/battenshed/innerrun.go`, `haltedStep`; `contracts/recipes/batten-recipe.yaml` header), so "its run is halted" means batten itself is paused or failed, not that the child halted.
The usual darn halt, a spent verify budget, therefore keeps the claim with a warning.
That outcome is listed in Open risks, but D10's case list does not show it.

**Fix:** Before removal, judge a landing from the child run's own status, for example a `Finalize` Done in its history, and mark such an entry done on every route.
Otherwise state in D10 that the clear lets a landed but unmarked run through, and what bounds it.
In D10, state that batten reads as halted only when batten itself is paused or failed.
Add an acceptance-criterion row for a landed but unmarked run removed by hand.

## Settled focus questions with no finding

- The `PR-Gate` to `Darn` bound in D3 matches the engine.
  `episodeStuckCount` (`internal/shedengine/run.go:515-538`) ends `PR-Gate`'s episode at a Done by any other producer in its segment (`segmentEnders`), and `Darn` shares `PR-Review`.
  `PRGate` returns Stuck only for a matching rejection (`internal/landingshed/prgate.go:46`).
  So each traversal is one operator rejection and `max_bounces: 5` is never reached, as D3 states.
- Vocabulary: the planned `landingshed` (D9) and `loomcli` (D11) texts name "the parent pair", its run records and absolute paths, never a side.
  That agrees with the record's `PATTERN-fabric-vocabulary` constraint.
- Config readers: D6's single reader covers the gate, Publish and Finalize alike, since all three call the one told closure, and an acceptance criterion pins the stop at each.
  F2 and F3 cover the gaps.
