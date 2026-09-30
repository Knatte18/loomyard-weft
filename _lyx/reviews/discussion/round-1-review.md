---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D3, D4, D5
    summary: The record never pins when the CLI loads logger.yaml relative to the arm check, and LoadOrTemplate's degrade path logs at Info, so an unconditional load arms the sink in every process of a worktree without logger.yaml and contradicts D3's never-armed-sweeps-nothing rule
  - id: F2
    severity: LOW
    location: _lyx/discussion/decision-record.md Goal
    summary: The Goal states that the step's trace_file must still exist when the driver reads it, but D1-D4 only keep it until N newer traces become active or the age bound passes, so the Goal claims an absolute guarantee the decisions do not give
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md D6
    summary: D6 restates Scope's out-of-scope line on arm triggers and level policy, and its rationale is the rejected suppression option's deliberation, already in support-log.md's Rejected alternatives and Interview item 5
  - id: F4
    severity: NIT
    location: _lyx/discussion/decision-record.md D1 rationale
    summary: D1's rationale about a single step spawning many lyx children reads as the Webster step, whose tmux children do not share its trace id (D2), so it leaves unsaid which children trace grouping actually serves
  - id: F5
    severity: NIT
    location: _lyx/discussion/decision-record.md Open risks
    summary: The Windows proc.IsAlive bullet declares itself safe and noted for completeness, so it is not a risk the plan writer acts on
  - id: F6
    severity: NIT
    location: _lyx/discussion/decision-record.md Auto-mode assumptions and Open risks
    summary: The unreproduced operator-surface cause appears both as an Auto-mode assumption and as the first Open risk, which already states the assumption and its consequence
  - id: F7
    severity: NIT
    location: _lyx/discussion/decision-record.md Open risks
    summary: The reconcile bullet is a requirement already carried by Scope and an acceptance criterion, not a risk
---

### [BLOCKING] Config-load timing is unpinned, and the degrade log can arm the sink

**Location:** `_lyx/discussion/decision-record.md` D3, D4, D5.

**Issue:** `configengine.LoadOrTemplate` logs `configengine: degrading to embedded template` at Info whenever the config file or `_lyx/` is absent (`internal/configengine/config.go:147`).
Any Info+ record arms the durable sink (`durableHandler.Enabled` accepts Info and above, `internal/logger/logger.go:302`; `durableWriter` calls `ensureDurableSink`).
Every existing worktree lacks `logger.yaml` until reconcile runs, and D4 treats the absent-file path as the normal template-defaults case.
The decision record never says whether the CLI loads the config in every process or only in one whose sink already armed.
D3's rationale ("at exit the CLI layer has already resolved the retention config") reads as a load before exit, for example in the root `PersistentPreRunE`.
Built that way, every lyx invocation in such a worktree arms a sink, writes a trace file, and sweeps at exit, including read-only commands that log nothing today.
That breaks D3's "a process that never armed a sink sweeps nothing", the matching acceptance criterion, and Scope's "the arm triggers stay as they are",
and it multiplies the routine trace files the task exists to stop crowding out run traces.
D5's Warn record arms the sink the same way; its rationale ("the Warn lands in the already-armed trace file") holds only if the load is gated on an armed sink, which nothing states.
The redirected-sink case has the same gap: D4 says such a process uses template defaults but not that it skips the load.

**Fix:** State the exit ordering in D3: `NotifyExit(code)` runs first, the CLI then asks the logger whether its sink armed in the current generation, and it loads `logger.yaml` and sweeps only when it did.
A redirected sink skips the load and uses template defaults.
Add a constraint that the config load adds no arm trigger: it never runs in a process whose sink has not armed.
Add an acceptance criterion: a process that never armed, in a worktree whose `logger.yaml` is absent or invalid, writes no trace file.
Reword D5's rationale to rest on that gate.

### [LOW] The Goal claims an absolute guarantee

**Location:** `_lyx/discussion/decision-record.md` Goal.

**Issue:** The Goal says a shed step's `trace_file` "must still exist when the driver reads it after the step returns".
After the step exits it loses its live-pid exemption; it then survives only while fewer than N traces have newer activity and its activity is younger than the age bound.
The ly-drive driver runs in a tmux-spawned Claude session, where `LYX_TRACE_ID` is stripped (`internal/reedengine/lifecycle.go` `stripTraceID`), so the driver holds no live pid in the step's group.
With N = 200 and the observed turnover of about 50 traces per 25 minutes, the window is ample, but it is a bound, not a guarantee.
The acceptance criteria pin the Webster-step ranking as a `Sweep` unit case, which is the right test for what the decisions actually deliver.
A plan writer who takes the Goal literally may reach for a guarantee mechanism nobody decided on.

**Fix:** Reword the Goal's second sentence so it states the bound: the step's trace file outranks every trace whose last activity is older, and so survives until N newer-active traces exist or the age bound passes.

### [LOW] D6 duplicates Scope and carries rejected-alternative deliberation

**Location:** `_lyx/discussion/decision-record.md` D6.

**Issue:** D6's decision ("arm triggers and the level policy stay unchanged") is already stated in Scope's out-of-scope list.
"Routine commands keep writing a trace file when they log at Info+" follows from unchanged arm triggers.
"'Fewer files from routine commands' is met by D1–D4" maps the board task's wording to decisions, and support-log.md's Interview item 5 already records it.
The rationale argues against the rejected suppression option; support-log.md's first Rejected alternative carries the same argument.
Completeness-before-leanness check: the only constraint D6 carries, unchanged arm triggers, lives in Scope, and F1's fix adds the related "config load adds no arm trigger" constraint.
Removing D6 loses nothing the plan writer needs.

**Fix:** Delete D6 from decision-record.md.

### [NIT] D1's rationale leaves unsaid which children grouping serves

**Location:** `_lyx/discussion/decision-record.md` D1 rationale.

**Issue:** "Counting files lets a single step that spawns many lyx children consume most of the budget" sits next to D2's Webster-step story, whose children arm "without an inherited trace id" and so each form their own group.
For that scenario grouping changes nothing and D2 alone carries the fix.
Grouping does serve lyx children launched by direct exec, which inherit the exported `LYX_TRACE_ID` (fabric and batten spawns use `os.Executable()`), while tmux-spawned sessions have it stripped.
The rationale is not wrong, but a reader can take it as the Webster fix.

**Fix:** Name the children in D1's rationale: direct-exec children that inherit the exported trace id, not tmux-spawned agent sessions, which D2 covers.

### [NIT] The Windows `proc.IsAlive` bullet is not a risk

**Location:** `_lyx/discussion/decision-record.md` Open risks, third bullet.

**Issue:** The bullet says the behaviour "errs toward keeping files — safe for this task, noted for completeness".
The plan writer has nothing to act on.

**Fix:** Delete the bullet.

### [NIT] The unreproduced operator-surface cause is stated twice

**Location:** `_lyx/discussion/decision-record.md` Auto-mode assumptions, last bullet; Open risks, first bullet.

**Issue:** The Open risk already says the failure was not reproduced, what other cause might apply, and that D2's acceptance test pins only the retention path.
The Auto-mode bullet repeats the same assumption; support-log.md's Q8 records it as an open question.

**Fix:** Delete the Auto-mode assumptions bullet and keep the Open risk.

### [NIT] The reconcile bullet is a requirement, not a risk

**Location:** `_lyx/discussion/decision-record.md` Open risks, last bullet.

**Issue:** "The plan must register the template wherever reconcile discovers module templates" is a requirement.
Scope already lists "reconcile registration", and an acceptance criterion requires `lyx config reconcile` to recognise and seed `logger.yaml`.

**Fix:** Delete the bullet.

## Focus verdicts

1. D3 + D5: a real gap, raised as F1.
2. Goal vs D1–D4: the decisions make survival likely by rank and count, not guaranteed; the acceptance criteria pin only the ranking unit case, which matches what the decisions deliver. Raised as F2 against the Goal's wording.
3. D1 vs D2: grouping does not help the Webster-step scenario; D2 carries it. D1 is not contradictory, since grouping serves direct-exec children. Raised as F4, a NIT.
4. D6: carries no decision beyond Scope. Raised as F3.
5. Open risks and Auto-mode assumptions: the Windows note (F5), the duplicated operator-surface entry (F6), and the reconcile requirement (F7) go. The mtime and daemon-sweep risks stay: both are real consequences of D2 and D3.
