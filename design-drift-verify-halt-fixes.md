# Drift-Verify halts: no false guard halt on record edits, and a short halt note with the way forward first

`drift-verify-halt-fixes` · Task · bug, enhancement, drift, loom · running · Drift-Expect

Bundles issues #146 and #147; both are Drift-Verify halts from task `dop-pumps-off-negative-spp`.

## 1. The judge guard halts on decision-record edits (issue #146)

### What happened

`lym loom` halted at `Drift-Verify` (state `blocked`, anomaly `escalation-to-human`) with:

```
decision halt: the judge session changed what it may not: records: src/csharp/NORCE.Models/_lym/discussion/decision-record.md was changed; the snapshot taken before it ran ... way forward: the parent or the operator resets a moved branch to its snapshot HEAD, restores a changed tracked file from its HEAD ... deletes an added path and the current capture's scratch directory ..., keeps the snapshot directory, which the resume compares against, then runs lym loom resume
```

The run was resumed and halted a second time with the same message.
The second halt names the same capture scratch directory (`.lym/drift/current/<timestamp>-<pid>`) as the first, although it came from a new trace file.
Both halts show identical history (8 entries, `Drift-Verify / stuck`).

### Why it is abnormal

- The changed path is the run's decision record, which the parent legitimately edits with `lym loom decision add` and `lym loom decision replace`.
  The judge-session guard cannot tell a parent edit made while the judge ran from an edit by the judge, so it can halt a run for a change the judge never made.
  The note gives no way to tell which actor changed the file.
- The recovery text asks the operator to restore files and delete a scratch directory by hand, then resume.
  A resume that re-halts on the same stale capture directory gives no signal whether the restore was applied or incomplete.

### What to do

- Record who changed the guarded path (judge session or another process) in the halt note, for example by comparing the file's mtime to the judge's start and end.
- Exclude the decision record from the guarded set, or snapshot it under a lock that `lym loom decision add` and `lym loom decision replace` honour.
- On resume, re-check the guard against the snapshot and say in the output which path still differs, rather than repeating the first halt's text.

Provenance: task `dop-pumps-off-negative-spp`; notes `loom-halt-2.md`, `loom-halt-3.md`.

## 2. The halt note repeats one rationale per row (issue #147)

### What happened

`lym loom` halted at `Drift-Verify` (state `blocked`, anomaly `escalation-to-human`) because the judge ruled that a change did not match its expectation.
The halt reason, which `lym loom` writes whole into the halt note under the friction directory, is one sentence per changed line of the compared capture.
It listed over a hundred changed lines (a ticked time series), each carrying the same judge rationale, and the note grew to about 44 KB.
The halt output ends with the way forward (`lym drift amend <slug> --entries <file>` then `lym loom resume`, or `lym loom goto --to Darn-Write`) only after all of that text.

### Why it is abnormal

- The same rationale repeats on every row, so the actionable information (the one cause and the two ways forward) is buried.
- A reader of the halt note or the pane has to scroll past the whole list to reach the way forward.
- The same text goes into every consumer of the note (reflection, operator pane), which inflates their context.

### What to do

- Group judge findings that share a case, key and rationale into one entry with a row range and the first and last old/new values.
- Cap the halt reason's length, point to the full findings file by repo-relative path, and put the way forward first.

Provenance: task `dop-pumps-off-negative-spp`; note `loom-halt-4.md`.
