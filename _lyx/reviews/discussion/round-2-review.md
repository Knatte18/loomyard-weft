---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:169
    summary: D9 does not say which stencil opens each role, though several D2 roles are built from several files or several stencils
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:122
    summary: D6 bounds an abandoned request by the handoff timeout but not a marker that outlives a dead or restarted watcher
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:40
    summary: D1 says the stencil carries the whole procedure, but the orch stencil defers board mechanics to ly:board, and the acceptance criteria and D7 restart rule disagree with the skip rule
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:186
    summary: D10 exempts the discussion role's interactive-mode rules from the scan, but those rules are Go text in modeRules, not a registered stencil
---

### [MEDIUM] Opening stencil per role is undefined for multi-file roles

**Location:** `_lyx/discussion/decision-record.md` D9 (line 169), D2 (lines 49-58), D10 (line 188).
**Issue:** D9 says a role's opening stencil is the one its module passes as the spec prompt, and a role built from a prefix or step files renders the directive "into the file that opens the session".
It never names that file for any role.
D2's rows span very different shapes.
Webster's master and recovery are `webster-template-master` and `webster-prefix-recovery`, and its forks and implementers are `webster-prefix-fork` and `webster-body-*`.
Burler is `burler-template-round-orchestrator` plus three `burler-step-*` files.
"Judge and triage" is three stencils (`treadle-template-judge-circling`, `-judge-milestone`, `-triage`).
Bouncer has `judge`, `seed`, `escalation` and `parent-notice`, of which only some are roles.
D10's first test reads the mapping, so a role left out of it passes the scan with no directive.
The registry is meant to hold this mapping, but the Decision fixes no rule for choosing the entry, and the plan writer must guess for the multi-stencil rows.
**Fix:** State the rule in D9, for example: per D2 row, the stencil the spawning module reads first for that row's session; a row with several stencils names each; step and body files never carry the directive.
State explicitly which D2 rows the rule covers.

### [MEDIUM] A request marker can outlive its watcher

**Location:** `_lyx/discussion/decision-record.md` D6 (lines 122-123), acceptance criteria (line 237).
**Issue:** D6 bounds a request with no live watcher (no marker is left) and a request whose note or idle probe does not pass within the handoff timeout (abandoned, marker removed).
The abandonment is done by the watcher, so it covers only a watcher that stays alive.
Two cases are open.
The watcher is live when the verb writes the marker and dies before the timeout; a later watcher then finds a marker with no stated age limit.
Or the watcher restarts mid-request, and the record does not say whether the timeout is measured from the request time (persisted in the marker) or restarts with the process.
A stale marker could then clear or compact a session long after the operator asked, which is the attack-surface question for a new verb.
**Fix:** In D6, state that the marker records its request time, that a watcher which finds a marker older than the handoff timeout removes it with a log entry and acts on nothing, and that the timeout runs from the recorded time across restarts.
Add a matching acceptance line.

### [MEDIUM] Skill-skip bound disagrees with the orch stencil, the acceptance criteria and the restart rule

**Location:** `_lyx/discussion/decision-record.md` D1 (line 40), D3 (line 76), D7 (lines 141-142), acceptance criteria (lines 229-230).
**Issue:** D1 states what a skipped skill lets through: the role runs on its stencil alone, "the stencil carries the whole procedure", so only style and quality guidance is lost, visible only in the log.
That holds for spawned roles but not for the orch.
D3 makes the orch stencil point at `ly:board` for board mechanics rather than restate them, so a skipped `ly:board` loses procedure, not style.
The bound D1 gives is therefore wrong for that one skill.
Separately, D7 says a restarted watcher re-types an unconfirmed skill load, while a skill that never ends a turn is skipped after the handoff timeout.
The record does not say whether a skip is persisted.
If it is not, each restart re-types the skipped skill and gives it a fresh timeout; if it is, "never loses one" in the acceptance criteria needs the skip as a stated exception.
The acceptance line "each end with the orch skills loaded" also contradicts the skip rule, since a skipped skill is not loaded.
D1 names a bound "the engine owns" and D7 uses the handoff timeout, so the two bounds are not stated as one.
**Fix:** In D1 or D7, state that a skipped `ly:board` leaves the orch without board mechanics, logged and visible to the operator, and name what bounds it (for example a notice queued through `orchengine` or a log-only record, whichever Discussion intends).
State that a skip is persisted in the phase so a restart does not re-type it.
Reword the acceptance line to "loaded, or logged and skipped on timeout", and name one bound for both D1 and D7.

### [LOW] D10's exception targets text no stencil scan sees

**Location:** `_lyx/discussion/decision-record.md` D10 (line 186), D9 (line 171).
**Issue:** The record does name the interactive discussion role as an exception, with a bound: the operator-ban line is dropped only there, and a blocked step still goes to the parent (Goal, D9).
That core question is settled, and it is not a defect.
But the interactive questions are produced by `modeRules` in `internal/loomengine/prompt.go` and injected through the `{{.mode_rules}}` marker, so the registered stencil holds no ask-the-operator text for the scan to exempt.
`PATTERN-parent-directive`'s "except the discussion role's interactive-mode rules" therefore names text that lives in Go, and the second test's exemption has no defined form.
**Fix:** In D10, say what the scan exempts: the `{{.mode_rules}}` marker in the discussion stencil, with the scan checking only stencil text, and that the autonomous `modeRules` branch is consistent with the directive.
Make the PATTERN entry's wording match.

## Focus departures

None.
