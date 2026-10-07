---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md Auto-mode assumptions, Open risks, D4 Bound
    summary: The 30-minute deadline assumption is false for the autonomous callers the hold is wired to, so the stated bound on an unnoticed held run is wrong
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks and D2 Bound
    summary: Open risks says the transcript fallback applies if the payload shape changes, but D2 skips the transcript whenever the key is present, and the D2 bound leaves non-running statuses and a non-list value unstated
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5
    summary: The hold notice routes agent-authored text into the orch session with no stated length bound or trust statement
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5 and D6
    summary: D6 leaves open the legacy asking record with no AskingOffset and whether the old ask is notified, and D5 leaves open whether the notified offset is persisted before or after the notifier call
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D9 and D5
    summary: D9 says the hold notice names the shell but D5 defines the notice content without the outstanding tasks
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D8
    summary: D8 does not say how ends with the tail is judged for trailing whitespace, so a sender that already ends with the tail can still get it twice
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Scope
    summary: Scope omits in-scope Decisions, namely the waiting trace line, the legacy asking attach and the AskUserQuestion steer text in settings
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Decisions D2 bound, D6 legacy cases, D8 stencil and steer text and D9 webster note and notice have no acceptance criterion
  - id: F9
    severity: LOW
    class: scope
    location: _lyx/discussion/support-log.md Rejected alternatives
    summary: Two requirements live only in the support log, that lyx never kills an expired shell and that forks never expire, and the plan writer never reads it
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D1 and D3
    summary: D1 and D3 carry incident narrative the plan writer does not need, which belongs in the support log
---

### [BLOCKING] The 30-minute deadline assumption is false for the callers the hold is wired to

**Location:** `decision-record.md` Auto-mode assumptions (fourth bullet), Open risks (second bullet), D4 Bound.
**Issue:**
D4 bounds a held run by its run deadline, and the assumptions call "the 30-minute default run deadline" an acceptable upper bound for a held run nobody answers.
30 minutes is only `shuttle.yaml`'s fallback `run_timeout_min`.
The autonomous callers D5 wires the notifier into set their own `Spec.Timeout` from their own config: `master_timeout_min` 480 (`internal/websterengine/template.yaml`), `discussion_timeout_min` 480, `review_timeout_min` 240 and `plan_timeout_min` 120 (`internal/loomengine/template.yaml`).
An agent held after a missed notice therefore waits up to 2 to 8 hours, not 30 minutes, and nothing else ends it.
The bound also leaves two cases unstated.
A standalone autonomous `lyx shuttle run`, which D7 says now prints `done`, `died` or `timeout`, and any spawner with no notifier, both used to end at once on an asking stop and now hold silently until the deadline.
`Attach` computes a fresh `now + Timeout` deadline on every attach, so a held run resumed after a driver crash gets a renewed deadline, and D4's "never extended by the hold" does not bound the cumulative wait.
**Fix:**
State the real limit: the deadline is the caller's own timeout from its config, with the per-caller values named as config keys rather than numbers.
Restate the assumption as an accepted bound on those deadlines, not on 30 minutes.
State that a no-notifier autonomous run, including `lyx shuttle run`, holds silently to its deadline.
State that each attach starts a fresh deadline.

### [MEDIUM] Open risks contradicts D2, and the D2 bound is incomplete

**Location:** `decision-record.md` Open risks (first bullet), D2 Bound, Auto-mode assumptions (third bullet).
**Issue:**
Open risks says that if Claude Code "drop[s] `background_tasks` or change[s] its shape; the transcript fallback then applies".
D2 applies the fallback only when the key is absent.
A changed entry shape under the same key, for example a renamed `status` field, is read as an empty list, and the transcript is skipped.
The D2 bound names only a task Claude Code omits.
It does not say what a payload entry with a non-`running` status counts as, or what a present key with a non-list value (null, an object) means.
The consequence is not stated either: an under-reporting payload makes a turn end count while work runs, which finalizes Done if every output file exists and otherwise holds.
The check that would catch it is only a hand-run live scenario, and no acceptance criterion ties to it.
**Fix:**
Rewrite the Open risk to match D2.
In D2, state that a present key with a non-list value counts as absent and falls back to the transcript.
In D2, state that only `running` entries count, and state the under-report consequence as the bound.
Add an acceptance criterion row for the non-list case and for a non-`running` entry.

### [MEDIUM] The hold notice carries agent-authored text into the orch with no stated bound

**Location:** `decision-record.md` D5.
**Issue:**
D5 adds a routing edge: the start of the held agent's last message, an agent-authored string, is typed into the orch session by the notice watcher.
The Decision says "newlines collapsed, truncated" but gives no truncation length, and does not say what the orch is to treat that text as.
`QueueNotice` already refuses a newline and a leading `/`, and a fixed notice prefix keeps the second from arising, but the Decision does not state either.
**Fix:**
State the truncation length, state that the line begins with the fixed notice prefix, and state that the quoted message is data reported by the agent and never an instruction to the orch.

### [MEDIUM] D6 and D5 leave legacy-attach and notice-persistence cases open

**Location:** `decision-record.md` D5 (last paragraph of Decision), D6.
**Issue:**
D6 attaches a legacy `asking` record "from its recorded `AskingOffset`".
An older binary's record can carry no `AskingOffset`: the field is optional and today's code treats nil as respawn-eligible, which is the #407 duplicate-agent path.
D6 does not say which way nil goes.
When the attach starts at `AskingOffset`, the old ask turn end is skipped, so no notice is sent for it, and the agent then holds silently to the deadline.
D6 says "attached as a held run" without saying whether that notifies.
D5 persists the last notified offset in `RunState` but not whether it is persisted before or after the notifier call, which decides whether a crash between them loses a notice or repeats one.
The acceptance criterion says "exactly once".
**Fix:**
State: a legacy record with a live strand and no `AskingOffset` attaches from the prompt offset, so its last turn end is read as a held turn end and notified once.
State: a record with an `AskingOffset` attaches after it and does not notify the old ask.
State: the notified offset is persisted before the notifier is called, so a crash loses at most one notice and never repeats one.
Align the acceptance criteria with these.

### [LOW] D9 says the notice names the shell, but D5's notice does not carry the outstanding tasks

**Location:** `decision-record.md` D9, D5.
**Issue:**
D9 says an expired shell's turn end is "a hold with its notice naming the shell".
D5 defines the notice as strand name, held state, message start and the way forward, with no outstanding tasks.
**Fix:**
Add the outstanding tasks, with their kind and label, to the D5 notice content, and name that in the notice-content acceptance criterion.

### [LOW] D8 leaves the already-ends-with-the-tail check unspecified for whitespace

**Location:** `decision-record.md` D8.
**Issue:**
`lyx shuttle send` appends the tail "unless the text already ends with it".
A sender that ends with the tail plus a trailing newline or space is not matched by a literal suffix test and gets a second tail.
The Decision states what the skip does, but not what bounds it.
**Fix:**
State that trailing whitespace is trimmed before the check and before appending, so a text that already ends with the tail is sent unchanged apart from that whitespace, and add it to the `lyx shuttle send` acceptance criterion.

### [LOW] Scope omits several in-scope Decisions

**Location:** `decision-record.md` Scope (In).
**Issue:**
The In list does not name the waiting trace line (D3), the attach of a legacy `asking` record (D6) or the autonomous AskUserQuestion steer text in `claudeengine/settings.go` (D8).
Each has a Decision and, for the third, a Constraints doc entry.
**Fix:**
Add one bullet for each.

### [LOW] Several Decisions have no acceptance criterion

**Location:** `decision-record.md` Acceptance criteria.
**Issue:**
No criterion pins the D2 under-report bound (see F2), D6's no-offset case and no-notice case (see F4), D8's stencil statements and the steer text, or D9's changed webster note and warning.
The D10 scenario list carries the `background_tasks` completeness check, but the only criterion on the live scenarios is that they compile.
**Fix:**
Add one criterion for each, as a behavior to cover, not a test to write.

### [LOW] Two requirements live only in the support log

**Location:** `support-log.md` Rejected alternatives; `decision-record.md` Constraints.
**Issue:**
Under the completeness-before-leanness test, "lyx does not own the agent's shells and never kills an expired one" and "forks do not expire like shells" are requirements the plan writer needs.
D9 and D3 imply them but never state them as constraints, and `Plan-Write` never reads the support log.
**Fix:**
Add both to `## Constraints` in `decision-record.md`, and keep the deliberation narrative in the support log.

### [NIT] Incident narrative in D1 and D3 belongs in the support log

**Location:** `decision-record.md` D1 (the SIGQUIT and hibernation sentence and the `continued-in` sentence), D3 Rationale.
**Issue:**
The operator's SIGQUIT, the host hibernating, `continued-in` being a consequence of the idle, and "#411 needed a goroutine dump" are incident history.
The plan writer needs the mechanism (queue-operation and `queued_command` lines, matched only on user messages), which stays.
**Fix:**
Move the incident narrative to the support log's Interview entry 1, and keep the mechanism and the two session references in D1.
