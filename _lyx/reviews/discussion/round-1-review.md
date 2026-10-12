---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md, D7 (inherited-slot exemption)
    summary: The inherited-slot exemption turns the watchdog off on every gated verify run, including the webster verify gate the incident happened in, because verifytree.Verify holds the slot and hands LYX_GATE_SLOT to the verify command, not an enclosing lyx gate test.
    fix: In D7, replace the inherited-slot exemption with one keyed on an enclosing `lyx gate test` itself, for example a variable that only gatecli sets in its own go test child's environment, so a `lyx gate test` run as a verify command inside a slot verifytree holds still runs its watchdog and only one nested under another `lyx gate test` skips it; restate the Decision's bound to match, correct the premise that the slot holder is always an outer `lyx gate test`, and add an acceptance criterion that a `lyx gate test` started with an inherited slot but no enclosing gate test runs the watchdog while one started under another `lyx gate test` does not.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, D7 (busy threshold) and the integration acceptance criterion
    summary: At least one full core in every sample is the ceiling for a single spinning goroutine, so tick granularity or contention for the slot's cores puts samples just under one core, restarts the window and lets a single-goroutine livelock run to go test's timeout; the 1-second integration test spinning one core is flaky for the same reason.
    fix: In D7, set the busy threshold below one core by a stated fixed margin (for example 90 percent of one core over the sample period) and say why; keep the sample-restarts-the-window rule against that threshold; update the Open risk about livelocks below one core to the new threshold; keep the integration criterion but have it spin one goroutine, so it pins that a single spinner is caught.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, D6 and the Open risk on a healthy over-window package
    summary: A watchdog quit of a healthy package ends the run stuck, and the Open risk says the operator's way out is raising busy_quit_sec, but D6's stuck reason and refusal-spec row only say the step hung and the stacks are in the log, so the refusal names no way forward for a false hang.
    fix: In D6, state that when the matched marker is D7's watchdog line, the gate report, the fixer's findings and the stuck reason name the window and say that a package that is CPU-bound but healthy needs `busy_quit_sec` in the hub's `gate.yaml` raised through `lyx config gate` from the prime; add that way forward to the refusal-spec row's change, and extend the stuck-reason acceptance criterion to a watchdog-quit hang naming the key.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, D5 (exit-code rule and its Bound)
    summary: D5's Bound covers only the text markers, but the exit code 128-or-above rule also reads a non-signal exit as a hang, such as git's fatal 128 or a tool's 255 in a hub's arbitrary verify shell command, and D6 then tells the fixer the step hung with stacks in the log.
    fix: In D5's Bound, add that any exit code of 128 or above reads as a hang whether or not a signal caused it, and that all it skips is the one rerun; in D6, have the report and findings name the exit code and say stacks are in the log only when a timeout or SIGQUIT marker matched, so a bare high exit code is reported as killed or exited with that code rather than as a hang with stacks.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md, D3 (map values)
    summary: D3 merges a map value by key with no unchanged check, so a value deleted and re-created, or replaced wholesale, under the same key keeps the old value's unknown keys, which contradicts D3's own rationale that a key is never carried onto a different element and is left unbounded.
    fix: In D3, state the bound for map values, that a value replaced under the same map key keeps the old value's unknown keys; add that case to the Open risks entry on a fresh record inheriting an old file's keys.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D9 and Scope Out (agent-written contract files)
    summary: D9's scan fails on any production DisallowUnknownFields call, which is broader than its rule about Go-written JSON records, and D9 does not say that the strict agent-written contract decoders Scope Out keeps are all yaml.v3 KnownFields and outside the scan, nor that a JSON contract file could not be decoded strictly.
    fix: In D9, add one sentence saying that the scan covers only encoding/json's DisallowUnknownFields; yaml.v3's KnownFields, which the strict agent-written contract decoders use, is outside it, and a future agent-written JSON contract that needs strictness validates its keys some other way.
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md, D7 (-bench and -fuzz exemption)
    summary: The -bench and -fuzz exemption states why it skips the watchdog but not what bounds a hung benchmark or fuzz run once the watchdog is off.
    fix: In D7, add to the Bound that a run carrying -bench or -fuzz is bounded only by go test's own -timeout, as before this change.
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, D7 Rationale (proc-child-lifetime sentence)
    summary: The sentence that the sampler joins internal/proc beside the child-lifetime code proc-child-lifetime plans to move there is board-fit deliberation; the placement it supports is already in the Decision text.
    fix: Delete that sentence from D7's Rationale and record it in support-log.md's Interview entry on board-fit; the Decision's placement of the sampling in internal/proc and the loop in internal/gatecli stays.
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md, before the D8 heading and before the Open risks heading
    summary: D7's last line runs straight into the D8 heading, and the last Auto-mode assumption runs straight into the Open risks heading, with no blank line, unlike every other heading in the file.
    fix: Insert one blank line before the D8 heading and one before the Open risks heading.
---

### [BLOCKING] Inherited-slot exemption disables the watchdog on every gated verify

**Location:** `_lyx/discussion/decision-record.md`, D7.

**Issue:** D7 skips the watchdog in a `lyx gate test` that runs in an inherited slot, on the premise that "the outer verb's tree already holds its test binaries".
The slot is not inherited only from an outer `lyx gate test`.
`verifytree.Verify` acquires a gate slot and runs the verify command with `lease.Env(env)` (`internal/verifytree/verify.go`, around the `slots.Acquire` call), and `Lease.Env` appends `LYX_GATE_SLOT` (`internal/gateslot/pool.go`).
The hub's verify command is `lyx gate test ./... && lyx gate test --tags integration ./...` (`_board/_lyx/config/darn.yaml`), and `internal/gatecli/run.go` takes the inherited branch whenever `pool.Inherited` holds.
So every `lyx gate test` the webster, loom and landing verify gates, `lyx webster verify` and card verifies run is in an inherited slot, with no outer watchdog, and D7 never fires on the path `board-field-drop`'s incident took.
The Goal's second half fails for the case the board entry names.

### [MEDIUM] One-full-core threshold misses a single spinning goroutine

**Location:** `_lyx/discussion/decision-record.md`, D7, and the integration acceptance criterion.

**Issue:** A single spinning goroutine uses at most one core; CPU time read from `/proc/<pid>/stat` comes in clock ticks, and the gate slot's `-p` cap runs other packages beside it.
Samples then land just under one core, and D7's "a sample under one core restarts that binary's window" keeps restarting.
The incident's 600% case is caught, but a single-goroutine livelock is not, and the acceptance test spinning "on a core" with a 1-second period sits on the same boundary.

### [MEDIUM] A false watchdog hang ends stuck with no way forward naming the key

**Location:** `_lyx/discussion/decision-record.md`, D6, and the Open risks entry on a healthy package past the window.

**Issue:** The Open risk says a healthy package past the window is quit on every attempt, the gate runs out of attempts and the operator raises `busy_quit_sec`.
D6's stuck reason says only that the step hung, was not re-run and the stacks are in the log, and the existing refusal-spec row's way forward is "fix the cause on the task branch".
For this case that way forward is wrong, and `PATTERN-refusal-way-forward` requires the refusal to name the real one.

### [MEDIUM] The exit-code hang rule has no stated bound

**Location:** `_lyx/discussion/decision-record.md`, D5.

**Issue:** D5 counts an exit code of 128 or above as a hang.
The verify command is any hub-configured shell command, and a process can exit 128 or above without a signal: git's fatal errors exit 128, and 255 is a common catch-all.
D5's Bound names only the text markers' false matches, and D6 then tells the fixer the step hung and the stacks are in the log, which is false for such an exit.

### [LOW] Map values carry unknown keys onto a replaced value

**Location:** `_lyx/discussion/decision-record.md`, D3.

**Issue:** D3 exempts map values from the unchanged check because "its key is its identity".
A map entry deleted and re-created, or overwritten with a fresh value, under the same key is a different value with the same identity, and it keeps the old value's unknown keys.
D3's rationale says a key is never carried onto a different element, and no bound or Open risk covers this case.

### [LOW] D9's scan is broader than its rule, and the exemption is unstated

**Location:** `_lyx/discussion/decision-record.md`, D9 and Scope Out.

**Issue:** The only production `DisallowUnknownFields` calls are `internal/state/state.go`, `internal/shedrun/seed.go` and `internal/battencli/cleanup.go`, all of which D1 and D4 remove, so the scan does not trip on the strict contract decoders today: those use yaml.v3's `KnownFields(true)` (`websterengine`'s outcome and report, `loomengine`'s arbiter, `planparser`, `shedbuild`).
D9 states neither that, nor that the scan forbids strictness for every JSON decode, agent-written included, which is wider than its own rule about Go-written records.

### [LOW] The -bench and -fuzz exemption has no stated bound

**Location:** `_lyx/discussion/decision-record.md`, D7.

**Issue:** The exemption removes the watchdog from a whole run; D7 gives the reason but not what still bounds a hung run of that kind.

### [LOW] Board deliberation in D7's Rationale

**Location:** `_lyx/discussion/decision-record.md`, D7 Rationale.

**Issue:** The `proc-child-lifetime` sentence argues placement from another board entry's plans.
The plan writer needs only the placement, which the Decision already states; the argument belongs in `support-log.md`.

### [NIT] Missing blank lines before two headings

**Location:** `_lyx/discussion/decision-record.md`, before `### D8` and before `## Open risks`.

**Issue:** Both headings follow the previous line with no blank line.

## Rejected

- D2 and D3 against the case-variant criterion, the opaque-type rule and the element-unchanged comparison: consistent.
  A case variant is known, so it decodes and is not kept; an opaque value is written whole and an element holding one compares by its encoding; a changed element drops its keys, which the acceptance criteria pin.
- D2's top-level and struct-field keeping going stale: bounded in D2 and listed in Open risks.
- D5's -1-with-empty-Detail rule: grounded; `verifyrun.Run` returns the shell's `ExitCode()` of -1 for a signal-killed shell with no error, and `verifytree.Verify` sets `Detail` only for a start failure or a timeout.
- D7's absent-versus-zero rule: bounded and pinned by an acceptance criterion.
- D7's SIGQUIT target selection: bounded to `.test` processes in the verb's own tree, once each, and the 15-character `comm` truncation is handled.
- Goal and Scope restating Decisions: Scope's In list points at Decisions by number rather than restating them, and the Goal states the outcome; no relocation needed.
