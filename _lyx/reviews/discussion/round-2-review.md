---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: decision-record.md D6 Bound and Acceptance criteria
    summary: D6 and D10 make llm compile-only, yet the Bound and an acceptance criterion require every retagged test to run under some gate, and the smoke-to-llm retag removes tests from any run with no stated bound
  - id: F2
    severity: MEDIUM
    class: consistency
    location: decision-record.md D6 Bound
    summary: Bound says every card verify runs -tags tmux, but card gates come from cardGateCommand, which is out of scope and runs only -tags integration
  - id: F3
    severity: MEDIUM
    class: design
    location: decision-record.md D2
    summary: D2 says a subprocess-spawning test is listed as no coverage but not how the tool detects the spawn, so a test that also covers in-process blocks can still be a prune candidate
  - id: F4
    severity: LOW
    class: consistency
    location: decision-record.md Constraints
    summary: Constraints cite PATTERN-test-time-seam, PATTERN-test-parallel, PATTERN-test-tier-purity and PATTERN-tmux-test-isolation, which D8 retires, so the plan writer is told to follow retired IDs
---

### [BLOCKING] llm tier is compile-only but the gate-coverage promise covers it

**Location:** `decision-record.md` D6 Bound; Acceptance criteria, the "Every test retagged out of `integration` or `smoke`" bullet.
**Issue:**
D6 moves every `smoke` file that spawns an LLM to `llm`, and D10 says `llm` is "compiled, never run (`go vet -tags llm <packages>`)".
The Bound still says "no test may end up under a tag no gate runs", and the acceptance bullet says every retagged test "still runs under some gate's verify … and no tagged test file sits under a tag no gate runs".
No gate runs `llm`, so both statements are false for the `smoke`-to-`llm` files.
The retag also lets through a loss: those tests leave any plan verify that ran `-tags smoke`, and nothing states that this is intended or what bounds it (`cardGateCommand` never ran `smoke`; it runs only `-tags integration`).
**Fix:**
State in the Bound that `llm` files are compile-checked by `go vet -tags llm` and run only by hand, that this is intended because the tier costs tokens, and that the compile check is what keeps them from rotting.
Narrow the acceptance bullet to files retagged into `integration` or `tmux`, and add that every `llm` file compiles under `go vet -tags llm ./...`.

### [MEDIUM] Card verify cannot be set to -tags tmux by this task

**Location:** `decision-record.md` D6 Bound.
**Issue:**
The Bound says "the plan verify and every card verify that touches a retagged test run `-tags tmux`".
Card gates are derived in Go by `cardGateCommand` (`internal/websterengine/cardgate.go`), which emits `go test -tags integration <dirs>`.
Scope puts that function out of this task, so the plan cannot make card verifies run `-tags tmux`.
Until `test-tiers` lands, Webster's card gates stop running every file moved to `tmux`.
**Fix:**
Say the plan verify (the final gate) runs `-tags tmux`, and that card gates lose the tmux tier until `test-tiers` decides, with the plan verify as the bound that still runs those tests.

### [MEDIUM] How a spawning test is recognised is not decided

**Location:** `decision-record.md` D2, the bullet on tests that spawn the built `lyx` binary or another subprocess; the matching acceptance criterion.
**Issue:**
Coverage cannot show that a test spawned a child, and a spawning test often also executes in-process blocks, so it has a non-empty set and can be a subset of others' union.
Without a stated detection rule, such a test is reported as a candidate while its child-side behavior is the only coverage of that path, which can send a prune task toward deleting the only test of a path.
The acceptance criterion cannot be checked without the rule.
**Fix:**
Decide the detection: a static scan of the package's test sources marks a test as spawning when it, or a helper it calls in the same package, references `os/exec`, `lyxbin`, `tmuxkit` or another testkit spawn helper.
The scan over-approximates, and a test it cannot classify counts as spawning, so the error direction is toward "no coverage", never toward "redundant".

### [LOW] Constraints name retired entry IDs

**Location:** `decision-record.md` Constraints, first and fifth bullets.
**Issue:**
D8 retires `PATTERN-test-tier-purity`, `PATTERN-test-time-seam`, `PATTERN-test-parallel`, `PATTERN-hermetic-git-tests` and `PATTERN-tmux-test-isolation`.
Constraints list them as the rules to follow, which sends the plan writer to IDs that will not exist.
**Fix:**
Name the surviving IDs (`PATTERN-test-speed`, `PATTERN-test-isolation`), noting the retired IDs only where the task starts from them.

## D5 verdict

Not a defect.
D5 requires a per-instance field or option, so a test can shorten the wait and keep `t.Parallel`.
`internal/reedengine/watchloop.go` already carries a `watchTiming` struct passed into the loop, so the watchdog interval has a per-instance carrier.
`reapExitTimeout` is a `const` read from free functions in `lifecycle.go`, `overlay.go` and `replace.go`, and `waitDelay` is a `const` inside a receiver-less `verifyrun.Run`.
Threading a field or option through them is plan-time call-site work, which the rubric excludes from this review.
D5's Bound states what a seam can change.
