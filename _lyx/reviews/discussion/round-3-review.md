---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 (hold notice line)
    summary: D5 bounds only the quoted message start, but the notice also carries outstanding task labels, which are raw agent-written shell commands with no control-character replacement or length cut, so a multi-line label makes QueueNotice refuse the line and the one notice is lost
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D5 and acceptance criterion on the notifier call
    summary: D5 says the fixed prefix labels the quote as the agent's words but states no delimiter around the quote, and the quote sits before the way-forward text in the same line, so agent text can run into the instruction part
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4 Bound (deadline sentence)
    summary: D4 says the held-run deadline is taken from the caller's own config, but several callers set Spec.Timeout from a --timeout flag, a recipe row timeout_s or a treadle profile value, so the stated source is false for them
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Auto-mode assumptions (first bullet) versus D5
    summary: The assumption lists loom, webster and burler as the notified spawners while D5 lists loom, webster, burler and batten-driven runs
---

### [MEDIUM] Task labels in the hold notice are an unbounded agent-text channel

**Location:** `_lyx/discussion/decision-record.md` D5, the sentence naming "the outstanding background tasks with their kind and label".
**Issue:**
D5 sanitizes only "the quoted message start": every control character replaced by a space, cut to 200 characters.
The same line also carries each outstanding task's label.
In `internal/shuttleengine/claudeengine/background.go` a shell's label is the entry's raw `command` (else `description`), untrimmed and uncut, and a Bash command can span lines.
`orchengine.QueueNotice` (`internal/orchengine/notice.go`) refuses any line containing `\r` or `\n`, and the notifier error is only logged at Warn (D5), so a held run whose outstanding shell has a multi-line command loses its one notice and holds silently until its deadline, the failure the notice exists to prevent.
The label is also agent-authored text reaching the orch's input with no length bound, outside the data-not-instruction label the quote gets, which the rubric's attack-surface question asks a Decision to bound.
The acceptance criterion on the notifier call pins the sanitizing of the quote only.
**Fix:**
D5 states that every agent-authored part of the line, the quote and each task label, has control characters including newlines replaced by a space and is cut to a stated character limit, with a cap on how many tasks the line names.
The acceptance criterion pins a held turn end whose outstanding shell has a multi-line command: the notifier is called once with a single-line notice.

### [LOW] The quote has no stated delimiter

**Location:** `_lyx/discussion/decision-record.md` D5, the sentence "the prefix labels the quoted message start as the agent's words, not a command".
**Issue:**
The decision says who owns the data-not-instruction claim: the notice's own label and the orch stencil's Messaging section, both stated (D5, D8, acceptance criteria).
It does not say how the quote is delimited.
The line order is strand name, held state, tasks, quote, then the way forward, so an undelimited quote can end mid-sentence into the way-forward text, or imitate it.
The acceptance criterion pins the fixed prefix but no delimiter.
**Fix:**
D5 states that the quote is enclosed in a fixed pair of delimiters and placed last, or the way forward is placed before it, and the acceptance criterion pins that the agent's text cannot appear outside the delimiters.

### [LOW] D4's deadline source is not true for every caller

**Location:** `_lyx/discussion/decision-record.md` D4 Bound, "taken from its own config ... each spawning module's `*_timeout_min` key".
**Issue:**
The per-module keys are right where they apply: `discussion_timeout_min`, `plan_timeout_min`, `review_timeout_min` and `friction_timeout_min` in `internal/loomengine/template.yaml`, `master_timeout_min` and `recovery_timeout_min` in `internal/websterengine/template.yaml`, `conflict_timeout_min` and `describe_timeout_min` in `internal/landingshed/template.yaml`.
The `run_timeout_min` fallback is right too, in `Spec.validate` and in `normalizeAttachSpec` for a zero `Spec.Timeout`.
But `lyx shuttle run` and `lyx burler run` take `--timeout`, a burler recipe row's `timeout_s` overrides `review_timeout_min` (`internal/shedrecipe/entries_burler.go`), and treadle passes a profile `Timeout`.
For those callers the deadline does not come from a `*_timeout_min` config key.
The fresh-deadline-per-Attach sentence is correct against `reconstructAndWait`.
**Fix:**
D4 states the deadline as the caller's own `Spec.Timeout`, however it sets it, with `run_timeout_min` where the caller leaves it zero, and drops "from its own config".

### [NIT] Notified-spawner set differs between D5 and the assumptions

**Location:** `_lyx/discussion/decision-record.md` Auto-mode assumptions, first bullet, and D5's wiring sentence.
**Issue:**
The assumption names loom, webster and burler agents; D5 adds batten-driven runs.
**Fix:**
Use one wording in both places, naming the group the wiring covers.

## Settled focus questions

These are verdicts, not defects.

- **Deadline bound per caller (focus 1).**
  Apart from F3, each stated bound holds against the code: the loom, webster and landingshed keys above, the shuttle fallback, and a fresh deadline per Attach.
  The loom driver spec leaves `Timeout` zero but is never waited on, so the hold does not reach it.
  Open risks names the autonomous run whose spawner wires no notifier and holds silently until its deadline, which the code confirms: only the loom, webster, burler and orch wirings build a runner, and the standalone `lyx shuttle run` has no orch.
  Not a defect.
- **Ownership of the data claim (focus 2).**
  Both the notice's own label and the orch stencil state it, and the acceptance criteria pin both.
  Control characters beyond newlines are covered for the quote, against `QueueNotice` refusing only `\r`, `\n` and a leading `/`, and the cut is stated in characters.
  The gaps are F1 and F2.
- **Legacy replay backlog (focus 3).**
  D5 and D6 decide one notice for the last held turn end of the replayed backlog, not one per turn end, and the legacy acceptance criterion pins it for a record with no `AskingOffset`.
  Not a defect.
- **Fallback wording (focus 4).**
  D2, Open risks and the acceptance criteria all state the transcript fallback as key absent or value not a list.
  Not a defect.
