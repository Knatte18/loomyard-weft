---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:91
    summary: "D4 in-flight route for a darn run with commits ends in goto --to Darn-Write after a step, which Shed refuses because goto only moves a run back; the route is also unneeded"
    fix: "In D4's last in-flight bullet, the acceptance criterion for D4 and the Open risks line, drop the split by commits past the branch point and the route goto Record-Write, step, goto Darn-Write, resume. Name one route for every run that reaches Darn-Write with no record: `lym loom goto --to Record-Write` then `lym loom resume`. State why it is safe, from internal/driftengine/doc.go: a run that has commits past the branch point passed Drift-Await, so its scope record is off or holds awaited_at, and Drift-Baseline and Drift-Await are then Done with no capture whatever HEAD is, and Drift-Baseline spawns its agent only when the scope record or baseline.md is missing. Darn-Write's missing-record Stuck then names that one route. If the writer keeps any multi-hop route, verify each hop against CheckGotoTarget and gotoAdmitted in internal/shedengine/goto.go, and cite the check."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md:93
    summary: "The new decisionWayNoRecord darn text tells a run parked past Record-Write to resume, the route D4 itself says points the drift agent at a missing record"
    fix: "In D4's paragraph on the missing-record refusal, make the darn text depend on the run's current row. At or before Record-Write it names `lym loom resume`; on any later row (an in-flight run parked on Drift-Baseline, Drift-Await or Darn-Write) it names `lym loom goto --to Record-Write` then `lym loom resume`, the route F1 settles. Add a clause to the D4 acceptance criterion so a test reaches the later-row text."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:97
    summary: "The differs-from-committed-copy way forward says restore the committed copy, which discards an entry decision add appended when its own commit failed"
    fix: "In D4's bullet for a record that differs from its committed copy, drop the premise that the difference is an edit outside `decision add`. internal/loomcli/decision.go also leaves an appended, uncommitted record when its commit fails (the refusal names `lym fabric commit`). State the way forward in two cases: when the working record differs from the committed one only by appended `Added after Discussion` entries or `## PATTERN changes` bullets, run `lym fabric commit` in the `_lym` weft repository, then re-run the verb; otherwise restore the committed copy. Extend the D4 acceptance criterion with the first case."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:203
    summary: "D13 places the load-time retired-key check in loom's and darn's config loads, but configengine.Layers already refuses the local file before module code runs, and PATTERN-config-layers bars a module from reading the layer"
    fix: "In D13, replace the sentence that the check runs in loom's and darn's config loads with a place that exists. Local-layer files are read and judged only by internal/configengine, and `Layers` refuses a non-None retired key there with the file and key but no way forward (internal/configengine/layers.go ValidateLocal, line 276). Give the Spec's `RetiredKeys` a per-key way-forward text that configengine's refusal appends, so the local-file load refusal names the file, the key and the local shed.yaml. State whether PrepareRunSettings reuses the same text (one source) or keeps its own check ahead of checkLocalDocument. Keep the stated position of the PrepareRunSettings check, after moveRunSettings and before module.Migrate, which matches internal/configreg/runsettings.go."
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:112
    summary: "D5 keeps Darn-Write's discussion gate as a re-prompting guard on a record D6 bars the writer from editing, and no Decision says how a present but damaged record is repaired"
    fix: "In D5, state what a failure of the Darn-Write validate-only guard does. Record-Write never overwrites an existing file (D4), so a present record or support log that fails the guard stays failing, and a re-prompt can only tempt the writer to edit what D6 forbids it to edit. Decide that a guard failure on this row halts the run Stuck with no re-prompt, the reason naming the findings and the way forward: remove the damaged file, then `lym loom goto --to Record-Write` and `lym loom resume` so Record-Write writes it again from the board entry (restoring the committed copy where the damage is uncommitted). Record that route in D4 beside the in-flight routes, and add it to the D5 acceptance criterion."
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:78
    summary: "D4 hand-rolls a temp file and rename and claims an existing file is always whole, but fsx.AtomicWriteBytes already does this, names temp files .tmp-, and does not sync"
    fix: "In D4, name `fsx.AtomicWriteBytes` as the writer of each file, so the leftover-temp cleanup has a known prefix (`.tmp-`, from os.CreateTemp in internal/fsx/fsx.go) and no second temp-and-rename is written. Scope the sentence that a file that exists is always whole to a process crash, since the helper does not fsync."
  - id: F7
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:73
    summary: "D4 and D5 claim CommonMark's fence rule but state only the length and character clauses; fencedLines also takes any line that starts with a fence run as a closer and ignores up to three spaces of indent"
    fix: "In D4, either add the two missing clauses to the rule fencedLines takes (a closing fence carries nothing but spaces after its run, so a line like ```js inside an open fence is body text; a fence may be indented up to three spaces) with a row in the D4 or D5 acceptance criteria, or reword D4, D5 and Scope to say the parser adopts the length and character clauses only, and drop the claim that an unclosed fence runs to the end of the document by CommonMark's rule for Discussion-written records. Record-Write's own fence is unaffected either way, since it is longer than every backtick run in the body."
---

### [BLOCKING] D4's in-flight route for a darn run with commits cannot run

**Location:** `_lym/discussion/decision-record.md:91`, repeated in the D4 acceptance criterion (line 258) and the Open risks line (251).
**Issue:** The route is `goto --to Record-Write`, `step`, `goto --to Darn-Write`, `resume`.
After `step` runs Record-Write, Shed persists the next row as the current one (`persistMarked` sets `CurrentProducer` to the routed row in `internal/shedengine/run.go`), here Drift-Baseline, state paused.
`goto --to Darn-Write` then asks for a row after the current one.
`gotoAdmitted` admits only rows at or before the current row, and `CheckGotoTarget` refuses the rest as "lies past the run's current row; goto only moves a run back" (`internal/shedengine/goto.go`).
The Darn-Write refusal would therefore name a step Shed rejects, which PATTERN-refusal-way-forward bars, and the acceptance criterion pins that text.
The detour is also unneeded.
A run with commits past the branch point went through Drift-Await, so its scope record is off or holds `awaited_at`.
Per `internal/driftengine/doc.go`, Drift-Baseline is then Done with no capture and Drift-Await is Done whatever HEAD is, and the agent spawns only when the scope record or `baseline.md` is missing.
`goto --to Record-Write` then `resume` reaches Darn-Write on its own.

### [MEDIUM] The missing-record refusal sends a run parked past Record-Write to resume

**Location:** `_lym/discussion/decision-record.md:93`
**Issue:** D4 says an in-flight run parked on Drift-Baseline or Drift-Await must be moved with `goto --to Record-Write`, and that a plain resume points the drift agent at a record that does not exist.
The new `decisionWayNoRecord` darn text says "resume the run past Record-Write with `lym loom resume`".
An orch that runs `decision add` first on such a run is sent down the route D4 calls harmful.
The refusal does not branch on where the run stands.

### [MEDIUM] Restoring the committed copy can discard an appended answer

**Location:** `_lym/discussion/decision-record.md:97`
**Issue:** D4 says a record that differs from its committed copy is "an edit outside `decision add`" and tells the caller to restore the committed copy.
`decisionVerb` in `internal/loomcli/decision.go` can itself leave the record appended and uncommitted: "the entry is appended but the commit failed", whose way forward is `lym fabric commit`.
The next `decision add` then reads the record as differing from its committed copy, and the new text tells the caller to restore the copy that lacks the answer.
The premise of the bullet is false for that case, and the way forward loses the parent's decision.

### [MEDIUM] D13's load-time check has no place to run

**Location:** `_lym/discussion/decision-record.md:203`
**Issue:** `loomengine.LoadConfig` and `darnengine.LoadConfig` call `configengine.Load`, which stacks the layers through `Layers`.
`Layers` runs `ValidateLocal` on the local file and refuses a non-None retired key, naming the file and key (`internal/configengine/layers.go` line 480), before any module code sees the document.
A check "in loom's and darn's config loads over the local layer" would have to read the local file itself, which PATTERN-config-layers reserves to `internal/configengine`, or it runs too late to see anything.
The missing piece is the way forward in configengine's own refusal.
The ordering question for `PrepareRunSettings` is settled and correct as written (see Rejected).

### [MEDIUM] The Darn-Write guard has no stated behaviour for a damaged record

**Location:** `_lym/discussion/decision-record.md:112`
**Issue:** D4 prevents a torn file and never repairs one: Record-Write writes only a missing file.
D5 keeps the `discussion` gate on Darn-Write, a gate that re-prompts the live session until its attempt budget is spent, and D6 bars that session from editing the record.
A record or support log that is present and fails the guard (an outside edit, or a filesystem that did not keep the rename whole) makes the writer retry a fix it may not make, or edit the record against D6, and ends in an exhaustion halt with no way forward.
The focus question on a truncated file is answered by "prevented, not repaired", and the recovery route is missing.

### [LOW] D4 re-specifies an existing atomic write

**Location:** `_lym/discussion/decision-record.md:78`
**Issue:** `fsx.AtomicWriteBytes` writes a `.tmp-*` sibling and renames it.
D4 describes the same mechanism in prose, leaves the leftover name its cleanup must match unstated, and claims an existing file is always whole, which a rename without fsync does not give across an OS crash.

### [LOW] D4's CommonMark claim is wider than the rule it states

**Location:** `_lym/discussion/decision-record.md:73`
**Issue:** `fencedLines` in `internal/discussionparser/append.go` flips on any line whose right-trimmed text starts with three backticks or tildes.
D4 adds the length and character clauses only.
CommonMark also forbids an info string on a closing fence and allows up to three spaces of indent.
The gap does not touch Record-Write's own body fence, but D5 relies on the CommonMark behaviour for an unclosed fence in a Discussion-written record.

## Rejected

- D4 fence length for a body with no backticks or inline code only: `max(3, longest run + 1)` gives 3 for both, a valid CommonMark opener, and an outer fence longer than every body run cannot be closed by a body line, so `fencedLines` treats the whole body as fenced.
- `decision add`, `locateOpenRisks` and `CheckDecisionIDs` reading only the record's own headings: `decisionsInsertOffset`, `locateOpenRisks` and `CheckDecisionIDs` already skip fenced lines through `fencedLines`, so the new closing rule reaches all three.
- `missingSections` as the one required-heading reader that ignores fences: it is the only one; the other readers in `internal/discussionparser` use `fencedLines`, and `relevant.go` reads the relevant list, not the record.
- D13 ordering in `PrepareRunSettings`: the check slots between the `keys` assertion and `module.Migrate`, loom's `retiredLoomKeys` drop and darn's `MigrateConfig` never list the new keys, and a hub base value is dropped by `dropRetired` on every load and pruned by reconcile as a unit the template lacks, as D13's Bound says.
- D6 and D7 with no parent and an answering `Decision for <slug>:` note: D6 and its acceptance criterion settle it, since the writer stops again until `decision add` puts the answer in the record.
- D4 route with no commits (`goto --to Record-Write` then `resume`): `goto` moves back from Drift-Baseline or Drift-Await, and Drift-Baseline keeps the scope record and a passed capture, so the claim holds; F1 extends it to the other case.
