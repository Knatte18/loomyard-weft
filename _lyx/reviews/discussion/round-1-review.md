---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:52
    summary: D3 sends the driver to envelope_path for friction_dir, scratch_dir and next_interrupt_policy, but the D2 fallback envelope carries no envelope_path and holds those keys inline, and neither D3 nor the acceptance criteria say what the driver does then
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:67
    summary: D3 downgrades the structural subshell guard on the drive directory to a prose no-cd rule without stating what a bare lyx call from a drifted cwd can do or what bounds it, and the only statement of that consequence sits in Open risks
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D2 Rationale says friction_dir and scratch_dir are needed only at parking or repair, but the driver stencil writes its stop report at every stop including done and busy, which lists friction_dir
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:119
    summary: D2 states the full-envelope fallback for a failed record write as well as for an empty StepsDir, but the acceptance criteria cover only the empty StepsDir case
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:33
    summary: Claude Code internals are narrated in D1 Rationale, restated in Auto-mode assumptions and again in the last Notes bullet, and Rationale carries a BASH_ENV comparison that is a rejected alternative
---

### [MEDIUM] Fallback envelope has no `envelope_path`, but D3 routes three keys through it

**Location:** `_lyx/discussion/decision-record.md:52` (D2) and `:68` (D3).
**Issue:** D2 prints the full envelope on stdout when no record is kept or the record write fails, and the full envelope is "unchanged in key set", so it has no `envelope_path`.
D3 rewrites `## Error envelopes`, `## Parking` and `## Repair records and the stop report` to say `friction_dir`, `scratch_dir` and `next_interrupt_policy` are "read from the file at `envelope_path`".
In the fallback case there is no such key and no such file, while the three keys sit in the printed envelope.
A driver following the D3 text literally has nothing to read.
Nothing in the acceptance criteria pins the stencil's handling of this case.
**Fix:** D3 states that when the printed envelope carries no `envelope_path`, those keys are read from the printed envelope itself.
An acceptance bullet requires the stencil to say so.

### [MEDIUM] Drive-directory guard becomes a prose rule with no stated bound

**Location:** `_lyx/discussion/decision-record.md:67` (D3), `:108` (Open risks).
**Issue:** The old rule (`(cd <drive-dir> && lyx …)`) made the right cwd structural for every call.
D3 replaces it with "never change the session's own cwd", which is a guard that depends on the driver's discipline.
The Bash tool keeps its cwd across calls, which is why the subshell existed and why the support log rejects `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` as out of scope, so drift is a live possibility.
The Decision states no bound for a bare `lyx shed step <run-id>` issued from another directory.
Open risks says the call "gets the recipe's own refusal text", but D3 does not say that, and it does not say whether a drifted cwd inside another seeded worktree can step a different run.
The old sentence "a wrong directory surfaces as the recipe's own refusal text, which you report verbatim" is neither kept nor dropped by D3.
**Fix:** D3 states what a bare call from a non-drive cwd can do (refuse through the missing-seed or ownership refusal, or address the run seeded at that location) and what bounds it, and says whether the verbatim-refusal sentence stays.
Open risks then points at that bound instead of being its only statement.
An acceptance bullet requires the stencil to contain the no-`cd` rule, since the existing bullet pins only the absence of the old form.

### [LOW] D2 Rationale understates when `friction_dir` and `scratch_dir` are read

**Location:** `_lyx/discussion/decision-record.md:58`.
**Issue:** "Needed only at parking or repair" is narrower than the driver stencil, whose stop report "lists `friction_dir`" at every stop, `done` and `busy` included.
D3 already lists the stop report among the sections that read from `envelope_path`, so the Decision is right and the Rationale contradicts it.
**Fix:** Say the keys are needed at a stop or a repair, not on a continuing step.

### [LOW] Record-write-failure fallback has no acceptance bullet

**Location:** `_lyx/discussion/decision-record.md:119`.
**Issue:** D2 and its bound ("a short envelope never names a missing file") cover both an empty `StepsDir` and a failed record write.
The acceptance bullet covers only the empty `StepsDir`.
The record-write-failure path is the one where the short envelope would name a missing file if the ordering were wrong.
**Fix:** Extend the bullet to "no record kept (empty `StepsDir`) or the record write fails".

### [LOW] CLAUDE_ENV_FILE internals narrated in three places; rejected-alternative clause in a Decision

**Location:** `_lyx/discussion/decision-record.md:33`–`37` (D1 Rationale), `:99` (Auto-mode assumptions), `:129` (Notes for the plan writer).
**Issue:** The prelude shape and the "read at session start, pushed ahead of the eval, same shell" claim appear in the Rationale, again in the Auto-mode assumption, and again in the last Notes bullet.
The `stdin: "pipe"` / `< /dev/null` mechanics and the "Unlike `BASH_ENV`" sentence are deliberation narrative: `support-log.md` Interview 2–3 and Rejected alternatives already hold them, and `## Decisions` is Decision + Rationale only.
Completeness-before-leanness check: the plan writer needs (a) the exact prelude shape to build the integration test, which the Notes bullet carries and Acceptance 3 names, and (b) that a child `bash` keeps its piped stdin, which Acceptance 3 pins.
Neither requirement lives only in the lines proposed for removal, so nothing is lost.
Evidence from the installed binary (2.1.289): the session environment loader reads `process.env.CLAUDE_ENV_FILE`, trims the content and puts it first in the script list, and caches the result with an explicit invalidation, so "read once per session" is slightly too strong; the premise itself holds.
**Fix:** Keep the premise once, in Auto-mode assumptions, worded without "once" and pointing at the Open risks entry as its failure mode.
Keep the exact prelude shape in the Notes bullet.
Cut the `stdin: "pipe"` mechanics and the `BASH_ENV` clause from D1's Rationale down to why the file works; the narrative stays in the support log.

## Focus verdicts

- D1 premise: supported by the 2.1.289 binary, which reads the variable from the process environment and prepends its content to the shell prelude.
  Hooks receive their own `CLAUDE_ENV_FILE` pointing elsewhere, so an operator hook cannot write into `bash-env.sh`; the bound "one `exec </dev/null` and nothing else" holds.
  The one over-claim is "once" (F5).
- D1 and D2 bounds match what the Decisions introduce, including the overridden operator variable.
  Not a defect.
- D2 against Acceptance: every key the driver stencil branches on per step is in the short envelope.
  The keys the stencil reads only at a stop come from the record, which is F1 and F3.
- D3 drive directory: F2.
- No Go code parses `lyx shed step` stdout for the removed keys: confirmed for production code.
  Tests that do (for example `internal/shedcli/parity_test.go` reading `scratch_dir` from a busy refusal) fail loudly under `go test ./...`, which the last acceptance bullet requires, so the assumption cannot fail silently.
  Not a defect.
- Relocation check: F5.
