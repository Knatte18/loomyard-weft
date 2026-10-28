---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/decision.go:775
    summary: preRun now builds the geometry and program name eagerly, so a failure there aborts every decision add, including ones that notify nobody
    fix: In preRun, do not abort on a ReedGeometry or FormatProgram error; leave programName empty or resolve it lazily inside notifyLiveRuns, and refuse only when a notice is actually needed and the name is unavailable.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/creelengine/deliver.go:64
    summary: probe reports pending for a strand with neither the CreelHooks mark nor PaneMail, so its worker keeps probing for as long as mail stands unread
    fix: In probe, return false, false when the strand has no CreelHooks mark and PaneMail is false, since nothing will ever be delivered to it.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: tools/sandbox/SANDBOX-CREEL-SUITE.md:115
    summary: Scenario S9 still shows the typed line with the old unquoted sender shape
    fix: Change the S9 line so the sender is quoted as in MessageLine, writing the quoted shortname and EXTERNAL name after the word from.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/creelcli/hookwait.go:34
    summary: hook-wait's never-exit-2 rule for an unreachable daemon or an exhausted reconnect budget has no test row
    fix: Add a row to the cli hook-wait table that runs the waiter against no daemon and asserts exit 0 with empty stdout and stderr.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/session.go:1
    summary: Nothing pins that a lym shuttle open session carries no append-system-prompt file (D9)
    fix: Add an assertion to the session-open launch tests that the opened session's command line holds no --append-system-prompt-file.
---

### [LOW] preRun resolves geometry eagerly in decision add

**Location:** internal/loomcli/decision.go:775
**Issue:** `preRun` calls `hubgeom.ReedGeometry` and `agentname.FormatProgram` for every call and aborts with exit 1 on failure. Before the diff, geometry was read only inside the arbiter closure for a named caller. A failure now blocks recording a decision that has no notice to send.

### [LOW] Worker keeps probing a strand that can never be delivered to

**Location:** internal/creelengine/deliver.go:64
**Issue:** `probe` returns `(false, true)` for a strand with neither mark, for example a Claude strand launched before the deploy. The worker polls it at the backoff ceiling for as long as mail is unread and delivers nothing. The cost is small and bounded.

### [LOW] Stale sender shape in the sandbox suite

**Location:** tools/sandbox/SANDBOX-CREEL-SUITE.md:115
**Issue:** The diff edited this file's header but left S9's line unquoted, while `MessageLine` now renders `from "<sender>"`.

### [LOW] Unreachable-daemon exit of hook-wait is untested

**Location:** internal/creelcli/hookwait.go:34
**Issue:** D4 says a waiter that cannot reconnect exits 0 and never 2. The cli table covers a subagent, an unknown engine and non-JSON input, so a change to exit 2 on the unreachable-daemon branch would pass.

### [LOW] No test that lym shuttle open stays without the trust sentence

**Location:** internal/shuttleengine/session.go:1
**Issue:** D9 says a `lym shuttle open` session carries no sentence. The launch-line and `Prepare` tests do not assert the absence on the session-open path.

## Rejected

- Lens code-tests: the init check on the trust sentence has no test. It is a package-load guard, and the file-content test already pins the sentence's text.
- Lens code-tests: a stderr write failure in `hookWait` is untested. It is an internal branch, below the public surface.
- Handler: `maxBodyBytes` stays 64 KiB where the second parent amendment says to lower it. Card 4 records that the line cap meets the amendment's intent and keeps the cap for replies to EXTERNAL, as D1 states.
- Handler: `TurnEndedAt` has second precision while a ring time has finer precision. A woken turn cannot end within the same second as its ring, so no ring is lost in practice.
- Handler: the directive declares no PATTERN changes, so nothing is owed under "Declared PATTERN changes landed".
