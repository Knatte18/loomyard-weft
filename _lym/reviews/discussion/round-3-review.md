---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:39
    summary: "Decision 1 detects a removed worktree by its root being gone, but the dobby's own state reads and writes recreate that root, so the exit can be defeated and its premise is false"
    fix: "In Decision 1, replace the sentence that says a gone worktree today only fails each tick with the fact that every `internal/state` read and write calls `os.MkdirAll` on the record's directory, so a tick after a removal recreates `<worktree>/<anchorRel>/.lym/beam` and with it the removed root. Then change the detection: the dobby tests, at the top of every loop iteration and not only on a probe tick, a marker its own writes cannot recreate, namely the worktree's git link at `Geometry.WorktreeRoot` joined with `.git`, and ends when that is gone. Say the in-memory `State.Tidy` the exit stops is the last non-nil one read while the marker existed, since a tick that runs after the removal reads a fresh empty `State`. State the bound that a tick already in flight when the worktree is removed can leave an empty `.lym/beam` stub, and that the exit's own writes create none. Add an Acceptance criterion: a beam dobby woken by a file event, not a probe tick, after its worktree is removed still exits, and a tidy held in `State.Tidy` is stopped."
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:49
    summary: "Decision 2 defines the re-exec test with pid and start ticks and widens it to a ticks-less record only for the lock wait, so Decision 3's notices leave open whether a ticks-less record counts as a re-exec"
    fix: "In Decision 2, name the test once, for example as the re-exec reading: the record holds this pid and either this process's start ticks or none. Say which of its uses treat the no-ticks case as a re-exec (the lock wait, and every notice of Decision 3 that is gated on the test, since the first deploy of this change is exactly a no-ticks re-exec) and that only `recordStart` differs, giving the no-ticks record a full pass and writing the ticks. Add to the Acceptance criterion on the record with this pid and no start ticks that a refused re-exec or a wiring failure on such a record posts its notice."
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:101
    summary: "Decision 5 says only the holder of a file's own dobby lock rotates it and that two renames never race, then has a Windows spawner without the lock rotate dobby.out and admits the race"
    fix: "In Decision 5, scope the sentence that begins with Only the holder of a file's own dobby lock rotates it to dobby.log. In the Windows paragraph, state that the spawner rotating dobby.out is the one exception to that rule, that it is bounded by reading the lock free through Live, and that the rename race the Bound already names is what the exception leaves."
---

### [MEDIUM] Decision 1 detects a removed worktree by its root being gone, but the dobby's own state access recreates that root

**Location:** `_lym/discussion/decision-record.md:39`, with `internal/state/state.go:31` and `internal/state/state.go:96`, `internal/dobbyengine/loop.go:272`.

**Issue:**
Decision 1 says a probe tick that finds `Geometry.WorktreeRoot` gone ends the dobby, and that today a gone worktree only fails each tick.
Neither holds.
`state.WriteJSON` and `state.ReadJSON` call `os.MkdirAll` on the record's directory, and `tick` opens with `UpdateState(d.geom.StatePath, ...)`.
After the beam's worktree is removed, the next tick recreates `<root>/<anchorRel>/.lym/beam` and with it the removed root, writes a fresh `state.json` and lock file there, and succeeds.
The loop sleeps at least one probe interval between timer wakes, but a file-watch event wakes it sooner and runs a non-probe tick, as does a tick already in flight at the removal.
Either recreates the root, so the next probe tick's check finds it present and the exit does not fire.
Decision 1's goal, that no part of a removed worktree is recreated, is met only for the exit's own appends.
The same recreation also defeats the exit's tidy stop: a tick after the removal reads a fresh empty `State`, so the in-memory `State.Tidy` the exit relies on is only valid if it is kept from before the removal.
The Acceptance criterion for the worktree-gone exit tests only the clean case where a probe tick finds the tree gone first.

### [LOW] Decision 2's re-exec test is stated one way and widened for one use, leaving Decision 3's notices ambiguous for a ticks-less record

**Location:** `_lym/discussion/decision-record.md:49`, `_lym/discussion/decision-record.md:70`, `_lym/discussion/decision-record.md:225`.

**Issue:**
Decision 2 defines the re-exec test as this pid and this process's start ticks, then says a record with this pid and no start ticks reads as a re-exec for the lock wait.
Decision 3 gates three notices on passing Decision 2's re-exec test, without saying whether the widened case counts.
The case is not rare: the first re-exec onto the binary that ships this change always meets a ticks-less record, which is the deploy this task hardens.
An implementer could post no notice for a wiring failure, a refusal or a fatal `Run` error on that very deploy.
The Acceptance criterion for the ticks-less record covers the lock wait, the full pass and the written ticks, and not a notice.

### [LOW] Decision 5's only-the-lock-holder rule has an unstated exception on Windows

**Location:** `_lym/discussion/decision-record.md:101`, `_lym/discussion/decision-record.md:105`, `_lym/discussion/decision-record.md:112`.

**Issue:**
The rule that only the holder of a file's own dobby lock rotates it, so two renames never race, is stated without limit.
The Windows paragraph has a spawner rotate `dobby.out` when `Live` reads the lock free, which holds no lock, and the Bound says two such spawners can race and lose a `.1`.
The record is internally consistent only if the rule is read as covering `dobby.log` alone, and a reader taking it as written would either refuse the Windows spawn rotation or believe the race cannot happen.

## Rejected

- Decision 3's pre-`Run` refusal tree, Decision 5's fd reopen and Decision 10's check lock and `check.out` were checked against the code and the focus entries, and nothing in them is wrong or left open.
- The adopt template value `{tiers: [0], scope: all}` matches `finalize`'s template value in `internal/gateslot/template.yaml`, so Decision 9's premise that the hub's tuned value is copied only when tuned holds.
- `gatecli`'s test function takes the writers for its output and for the tier lines, so Decision 10's in-process run with its output on `suite.log` is feasible without a second parser.
- `internal/selfexec` imports only `internal/logger` beyond the standard library, matching the leaf constraint.
