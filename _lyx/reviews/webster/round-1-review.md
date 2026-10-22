---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: handler
    location: PATTERN.md:5
    summary: Commit 802135a01 shortened about ten unrelated PATTERN.md entries, dropping binding clauses that main still carries
    fix: In PATTERN.md restore the text main carries for the entries commit 802135a01 rewrote, keeping only the declared run-process-owner add (use `git diff main -- PATTERN.md` to list them), then keep the three declared entries. The entries are durable-vs-ephemeral-state, cliwire-sole-wiring, config-strictness, owner-machine, glyph-conversion-chokepoint, verified-tree, orch-pane-single-writer, pause-holds-prompts and wait-mark-display-only.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/runproc/end.go:195
    summary: End watches only a record's leader, so a group member that survives SIGTERM after the leader exits gets no SIGKILL and is not reported, and a zombie or absent leader sends SIGTERM to its group with nothing to escalate
    fix: In planRecords add the live processes that report a signalled group, with their start times, to that group unit's members, so runUnits waits on them, escalates to SIGKILL and reports them as survivors. Also let runUnits treat a unit whose group was signalled as signalled when its members are empty. Add a row to TestEnd in internal/runproc/end_test.go for a leader that exits on SIGTERM while a group member ignores it.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/runproc/end.go:244
    summary: A failed signal call is only logged, so the unit's members land in neither ended nor survivors and the caller sees success while the process lives
    fix: In runUnits keep the members of a unit whose signal returned an error in the watched set, so a process still running after the bound is reported in the SurvivorsError. Add a row to TestEnd for a signalProcess error.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: PATTERN.md:35
    summary: The overview lines of PATTERN-pair-teardown and PATTERN-leaf-packages are unchanged, so the declared texts live only in the pattern/ background files
    fix: Leave the overview line as it is when the project treats the overview as an index, or extend PATTERN.md:35 and PATTERN.md:106 with the declared sentences. Decide which and state it in the PATTERN.md header.
---

### [MEDIUM] Unrelated PATTERN.md entries shortened

**Location:** PATTERN.md:5 and the hunks of commit 802135a01.
**Issue:** `git show 802135a01 -- PATTERN.md` rewrites nine entries outside the three declared changes, and main and the merge base both carry the longer text.
Clauses dropped include "same subpath", "alone resolves target dir, repo root and state", "bar `runprofile`", "verdicts in `ReconcileOrigin`", "a ref's alphabet only `planparser`", "told its scope's pause hold", "via `Delivery` seam", "only orch runner reads uncovered" and "no Go decision reads them".
PATTERN.md is the authoritative overview, so the rewrite weakens binding rules with no card or declared change behind it, and it will conflict with or revert main's text on landing.
The background files still hold some of the detail.
The code-pattern-fit fork judged these a sync with main; `git show main:PATTERN.md` shows the longer text, so they are not.

### [MEDIUM] End does not follow a signalled group's other members

**Location:** internal/runproc/end.go:195 and planRecords near line 255.
**Issue:** A recorded group leader is the only watched member of its group unit, and a descendant in an already signalled group is skipped.
A leader that exits on SIGTERM while a group member ignores it passes `await`, so no SIGKILL goes out and the member is not a survivor.
A leader that is a zombie or gone leaves `members` empty, so `runUnits` returns at `len(signalled) == 0` after the group already got SIGTERM.
The end then reports success while the run's process lives, which the pair teardown and `reed down` rely on.

### [LOW] A failed signal is not reported

**Location:** internal/runproc/end.go:244.
**Issue:** `signal` logs an error and returns false, so the members of that unit are not awaited, escalated or reported.

### [LOW] Overview lines for two declared entries unchanged

**Location:** PATTERN.md:35 and PATTERN.md:106.
**Issue:** The declared pair-teardown and leaf-packages texts landed in `pattern/PATTERN-pair-teardown.md` and `pattern/PATTERN-leaf-packages.md`, and `PATTERN-run-process-owner` landed in both places.
The overview lines read as an index, so this is a consistency note and not a missing landing.

## Rejected

- Card 14 has no test (code-tests): `internal/tenderengine/resume_test.go:852` pins the processes part, orphans included.
- `ChildRecord` ignores the `proc.StartTime` error (code-correctness): the read fails only when the child is already gone, and then nothing is left to end.
- Step-loop kill through `proc.KillTree` bypasses `runproc.End` (code-affected-set): the declared pair-teardown text keeps the loop kill after the owned-process end, so it is the declared order.
- Unrelated PATTERN.md edits are a sync with main (code-pattern-fit): main carries the longer text, so the claim is false; kept as the first finding.
- No `Pattern-Departure:` trailer exists in the range, so nothing needed judging.
- Every `runproc.End` call passes `os.Getpid()`: `internal/pairteardown/teardown.go:554` and `internal/tendercli/deps.go:110,324`; `EndProcess` serves only the orphans at `deps.go:113`.

## Coverage limits

The forks read the diff selectively.
I did not open the card 3, 5 to 8, 16 and 17 code paths (detached spawn registration, deferral, lift spawns, hibernate confirmation, shuttle waiting turn) beyond `pauserecord/hold.go` and `tenderengine/resume.go`, so the verdict covers only what is above.
