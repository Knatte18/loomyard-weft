---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:43
    summary: D1 names a batten state halted that does not exist; the real halt states blocked and awaiting are never named, so a covered blocked or awaiting run's state cell is unspecified
    fix: In D1 (decision-record.md:43) replace the sentence with "A covered run whose batten state is `blocked`, `awaiting` or `failed` keeps its state, since the pause holds nothing there.", and in the second acceptance criterion (decision-record.md:143) replace "a halted or failed covered run" with "a `blocked`, `awaiting` or `failed` covered run". The legal states are the six in `internal/shedengine/status.go` (running, paused, done, blocked, failed, awaiting).
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:53
    summary: D2's held set (idle-stalled, idle-done, dead) omits the asking reading and the live held-turn wait marker that shuttle's own wait reading counts as waiting, so an asking advisor whose up line has not landed still defers the chair's notice to the 10-minute cap, and the Decision does not say whether that is intended
    fix: In D2 decide whether an advisor reading `asking` is held for the predicate. If yes, add `asking` to the excluded readings at decision-record.md:53, add it to the first clause of the third acceptance criterion (decision-record.md:144), and add one Rationale clause saying the set matches shuttle's waiting reading (asking or idle-stalled). If no, add one Rationale sentence saying why an asking advisor still defers the notice. Either way, state whether the held-turn wait marker matters to the predicate; if it does not, say the session reading alone decides.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:68
    summary: D3 grounds the absent size cap on shuttle's pre-write guard standing in front of every write, which is false for reed's name repair (it calls the paste path directly, behind reed's activity check), and a pre-write guard bounds nothing about size; the bound for the removed escape and dash guard is only implied
    fix: >-
      Replace the sentence at decision-record.md:68 with "Bound: the text reaches tmux only as stdin bytes of `load-buffer -`, never as an argv element, so no text can add, end or alter a command in the list; size is unbounded by design, and every caller is lyx's own code, shuttle's sends behind its pre-write guard and reed's name repair behind reed's activity check."
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:85
    summary: D4 forwards a global flag typed before the shortcut name only onto the last argv, so lyx -v prime runs orch start without -v; the Decision says the flag reaches the expansion without saying it skips every argv but the last
    fix: >-
      In D4 (decision-record.md:85) decide and state one rule. Recommended: every `-v`/`--verbose` word before the first `--` is forwarded to each argv, the rest of the words to the last argv only. Then add to the trailing-words acceptance criterion (decision-record.md:147) "`lyx -v prime` runs both `orch start` and `reed attach` with `-v`". If instead the last-argv-only behaviour is kept, say so explicitly in D4.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:137
    summary: The pre-run open risk says the persistent pre-run re-runs twice, but the shortcut command's own invocation already runs it once, so lyx prime runs it three times; and the risk is left as narrative with no constraint the plan writer can test against
    fix: Move the item from Open risks to Constraints, worded "The root's persistent pre-run (trace adoption, sink arming, network gate install, stencil seeding) runs once for the shortcut command and once per argv in one process, and each repeat is harmless; the plan confirms each." Keep a shorter Open risks line only if a hook is found not to be idempotent.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:83
    summary: D4's sentence on departing from the board's same-root wording is deliberation narrative; the requirement (a fresh root per argv) is already stated in the sentence before it, and the support log's interview item 7 already records the departure
    fix: Delete the sentence at decision-record.md:83 ("This departs from the board's wording ... the board asks for."). No requirement is lost; the preceding bullet line already states the fresh root per argv.
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:144
    summary: The D2 acceptance criterion ends with the garbled phrase an advisor whose run ended died does not count
    fix: At decision-record.md:144 replace "an advisor whose run ended died does not count" with "an advisor whose run ended does not count".
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:70
    summary: Q2 in the question ledger still says a running or starting covered run, missing the batten-paused covered run that D1 and Q10 now include
    fix: At support-log.md:70 replace "of a running or starting covered run (D1)" with "of a running, batten-paused or starting covered run (D1)".
---

### [MEDIUM] D1 names a nonexistent `halted` state

**Location:** `_lyx/discussion/decision-record.md:43`, and the second acceptance criterion at line 143.

**Issue:** The overview's state cell renders the batten status's literal `State`.
`internal/shedengine/status.go` declares six states: `running`, `paused`, `done`, `blocked`, `failed`, `awaiting`.
`halted` appears only in shedengine's prose as the collective term for blocked and awaiting.
D1 puts `halted` in backticks next to the literal states `running`, `paused` and `starting`, so the plan writer can read it as a seventh literal state.
That leaves `blocked` and `awaiting` without a stated rule, and `awaiting` is the planned hand-off halt that a hub pause will often cover.

### [MEDIUM] D2's held set differs from shuttle's own waiting reading

**Location:** `_lyx/discussion/decision-record.md:53`.

**Issue:** `internal/shuttleengine/agentwait.go` counts a run as waiting when its held-turn wait marker is live, or its session reads `asking` or `idle-stalled`.
D2 excludes `idle-stalled`, `idle-done` and `dead`, but not `asking`.
An advisor that stops on a question or permission prompt before the sender lands its up line therefore keeps deferring the chair's held-turn notice until the 10-minute cap.
That is the case the Goal's second follow-up names ("an advisor that is itself held").
The Decision may intend this, but neither the Decision nor its Rationale says so.
The D2 premise itself holds: in `internal/seatengine/work.go`, once the up line lands only a busy-on-turn reading counts, so the remaining hole is the up-line clause, as the Rationale says.
The session-state-error case reads the same in the Decision and the code: an erroring reading is neither held nor busy, so only the up-line clause can count that advisor.

### [MEDIUM] D3's no-cap bound rests on a false premise

**Location:** `_lyx/discussion/decision-record.md:68`.

**Issue:** `internal/reedengine/namerepair.go` calls `pasteSubmitLocked` directly.
`PATTERN-pane-text-injection` states the same thing: "The watchdog's `/rename` name repair has no shuttle run; it stands behind reed's own activity check".
So "shuttle's pre-write guard still stands in front of every write" is false.
Even where the guard does apply, it checks readiness, operator quiet and an empty box, not size, so it does not justify leaving out a cap.
The real bound for the removed `;` escape and `--` guard is that stdin bytes are never parsed as tmux commands.
The Decision implies this ("the text is no longer an argv element") but never states it as what the change can let through.
The D3 premise is sound otherwise: on this host's tmux 3.6, `set-buffer` with a 40 000-byte argument fails with `command too long`, and a 70 000-byte stdin `load-buffer -b y -` chained with `paste-buffer -p -d` in one invocation exits 0.

### [LOW] D4 forwards global flags only to the last argv

**Location:** `_lyx/discussion/decision-record.md:85`.

**Issue:** The shortcut takes no flags, so the root's `-v` count flag typed before the name arrives among the shortcut's words, and D4 appends all of them to the last argv.
For `prime`, `orch start` then runs without `-v` and only `reed attach` gets it.
`-v` sets process-global log verbosity in the root pre-run, and the command whose trace an operator most likely wants is the first one.
D4's wording, "reaches the expansion the same way", hides that the flag skips every argv but the last.
The rest of D4's attack surface is bounded.
Trailing words reach only commands the same binary already exposes, each through a full root with its own guards.
`--help`, `-h` and `--json` before `--` run nothing.
`--` and the words after it are forwarded.
The Decision and the sixth acceptance criterion agree on that interception.

### [LOW] Pre-run re-run risk is miscounted and left as narrative

**Location:** `_lyx/discussion/decision-record.md:137`.

**Issue:** The shortcut command is itself a child of the outer root, so the root's `PersistentPreRunE` runs once for it before any inner root runs.
`lyx prime` therefore runs the pre-run three times, not twice.
The hooks I read look tolerant: `logger.MintOrAdoptAndExport` uses a `sync.Once`, `logger.Arm` takes the sink lock and re-arms, and `pausecli.InstallNetworkGate` reinstalls a gate.
But the item tells the plan writer to "check each" without a testable rule to hold the plan to.
The D5 shortcut-file test, by contrast, is a Decision and an acceptance row, as the focus asked.

### [LOW] Board-wording departure in D4 is deliberation narrative

**Location:** `_lyx/discussion/decision-record.md:83`.

**Issue:** The sentence argues the choice instead of stating it.
By the completeness-before-leanness test, the requirement it carries is already on the line before it ("on a fresh root from the same factory"), and the support log's interview item 7 already records the departure.
Deleting it loses nothing the plan writer needs.

### [NIT] Garbled phrase in the D2 acceptance criterion

**Location:** `_lyx/discussion/decision-record.md:144`.

**Issue:** "an advisor whose run ended died does not count" has a stray word.

### [NIT] Stale Q2 answer in the question ledger

**Location:** `_lyx/discussion/support-log.md:70`.

**Issue:** Q2 lists only running and starting covered runs.
D1 and Q10 also cover a batten-paused run.

## Focus coverage

- D4: covered under F4 and F6 above; the interception rules agree between the Decision and the criteria.
- D3: covered under F3.
- D2: the premise and the session-state-error case are consistent (see F2); the gap is the `asking` reading.
- D1: `running`, `paused` and `starting` read the same in the Decision, the Scope and the criteria.
  The zero floor matches `statuspane.FormatMinutes`, which already clamps negative durations to `0m`.
  The hub-wide pause with no run in flight is out of scope in one place only, consistently.
  The defect is `halted` (F1).
- D5 and Open risks: the shortcut-file test is actionable; the pre-run risk is not (F5).
- Relocation: F6.
  Notes for the plan writer names files and functions but no signatures, interface shapes or `file:line` citations, so it stays within the discussion stencil's limits.
- Exclusion: every Scope In item and every acceptance criterion maps onto one of the Goal's five follow-ups or the same-commit docs rule; nothing to exclude.
