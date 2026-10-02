---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D1 Bound)
    summary: D1's bound is silent on shape mismatches, which refuse today and would now silently replace the operator's present value
  - id: F2
    severity: LOW
    location: _lyx/discussion/decision-record.md (D1 Bound)
    summary: D1 does not say what happens to a missing key inside an element of a present list of mappings
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md (Notes for the plan writer, first bullet)
    summary: The Reconcile pointer omits that its merged output prunes file keys the template lacks and that its added list over-reports list-element keys
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Goal, Auto-mode assumptions, Open risks)
    summary: The Goal promises no manual step, but every worktree spawned before landing needs one lyx ide spawn before D3 reaches the #340 path
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (D2, Constraints, Acceptance criteria)
    summary: D2 does not name its liveness test, so Windows parity rests on an unstated assumption and has no criterion
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (D4, D5)
    summary: The tracked-tasks.json branch is unspecified for settings.json and the exclude write
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md (D6, Scope Out)
    summary: D6's board exemption is bounded by ownership rather than by writer and location, and misdescribes the board as a separate repo
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: D8 has no checkable acceptance criterion
---

# Discussion review — round 1

Evidence: I read both target files, the discussion stencil, `internal/configengine/config.go`, `internal/yamlengine/reconcile.go`, every module template, `internal/ideengine/spawn.go`, `internal/vscode/config.go`, `internal/reedcli/add.go`, `internal/reedengine/strand.go` and `apply.go`, `internal/agentname`'s `Resolve`, `internal/orchcli/start.go`, `internal/boardengine/sync.go` and the relevant `CONSTRAINTS.md` sections.
I ran `yamlengine.MissingKeys` and `yamlengine.Reconcile` on four fixtures in a throwaway `go run` probe (since deleted):

| Fixture (template / file) | MissingKeys | Reconcile added / removed | Reconcile merged |
|---|---|---|---|
| `status_line: {format: x}` / `status_line:` (null) | `status_line.format` | `status_line.format` / `status_line` | `status_line: {format: x}` |
| `b: x` / `b: {c: 1}` | `b` | `b` / `b.c` | `b: x` |
| `l: [{p: 1, q: 2}]` / `l: [{p: 5}]` | `l[0].q` | `l[0].q` / none | `l: [{p: 5}]` |
| `a: 1, b: 2` / `a: 5, extra: 9` | `b` | `b` / `extra` | `a: 5, b: 2` |

### [MEDIUM] D1's bound misses shape mismatches

**Location:** `decision-record.md` D1, Bound.
**Issue:** Today a file whose value at a template key has a different shape from the template's refuses: `status_line:` with a null value where the template has a mapping reports `status_line.format` missing, and a mapping where the template has a scalar reports the scalar key missing.
D1 now fills every missing key-path, but you cannot merge a leaf into a scalar or null node, nor a scalar over a mapping, without replacing the operator's present value.
That contradicts D1's own "A key present with any value, empty included, keeps that value", and the reuse the Notes suggest (`Reconcile`) does replace it silently, as the probe shows (`b: {c: 1}` becomes `b: x`).
The guard downgrade therefore lets through a case its bound never names.
On the other focus cases: a present key whose value is null with a scalar template keeps its null, which the bound already covers ("any value"); a filled template value that is an env marker with a default resolves through `envsource.Build`, just as the absent-file fallback does, so that case is not a defect.
**Fix:** State in D1's bound that a key-path is filled only when the file lacks it and every ancestor present in the file is a mapping.
A key whose present value's shape differs from the template's (a scalar or null where the template holds a mapping, or a mapping where it holds a scalar) still refuses, naming the key-path.
Add an acceptance criterion for both directions.

### [LOW] D1 is silent on a missing key inside an element of a present list of mappings

**Location:** `decision-record.md` D1, Bound.
**Issue:** `MissingKeys` reports `l[0].q` missing for template `l: [{p, q}]` and file `l: [{p}]`, because `sequenceBasePath` recognises only a trailing index.
D1's list rule speaks only to length.
It leaves open whether the loader fills `q` into the operator's element, carries the list whole, or refuses; `Reconcile` carries the list whole yet reports `l[0].q` as added, so a fill built on it would log a key it never filled.
No template that loads through `configengine` holds a list of mappings today, so the case is latent.
**Fix:** State in D1's bound that a present list is carried whole and nothing is filled inside its elements; a template key missing inside an element still refuses, as today.

### [LOW] The Notes' Reconcile pointer omits two behaviours that contradict D1

**Location:** `decision-record.md`, Notes for the plan writer, first bullet.
**Issue:** `Reconcile` builds its merged document on the template, not on the file, so it drops every file key the template lacks (probe: `extra` removed).
That contradicts D1's "merges the missing leaf keys from the template into the file's document" and `docs/shared-libs/configengine.md`'s "Extra/stale keys are tolerated".
Its `added` list also includes list-element keys it does not fill (F2).
The Note warns only about list length, so a plan writer following it would ship both regressions.
**Fix:** Have D1 say that keys the template lacks stay as the file has them, and add that to the acceptance criteria.
Rewrite the Note to name both `Reconcile` behaviours a reuse must avoid.

### [MEDIUM] The Goal promises no manual step, but existing worktrees need one spawn

**Location:** `decision-record.md` Goal, Auto-mode assumptions (fourth bullet), Open risks (first bullet); `support-log.md` Q12.
**Issue:** #340's path is opening the prime's hub workspace in VS Code while `lyx orch` hosts the hub; CLAUDE.md's workaround says to open it from a plain terminal instead.
The hub workspace's folder-open task comes from the prime's `.vscode/tasks.json`, which only `lyx ide spawn` writes.
D3 and D4 therefore reach the #340 path, including a reopen from VS Code's recents, but only once a `lyx ide spawn` has run after landing.
Until then, every existing prime still runs the old chain, even though D10 deletes the #340 workaround in the same commit.
The Goal's "with no manual step after either" contradicts the record's own Open risk and Q12.
The Auto-mode assumption "The prime is opened through `lyx ide spawn <prime>`" also misstates the opening path: the issue is about the reopen, not the spawn.
**Fix:** Have the Goal say no manual step is needed beyond one `lyx ide spawn` of each worktree spawned before this lands.
Reword the assumption: only `lyx ide spawn` regenerates `tasks.json`, and a later open from VS Code, recents included, runs the last-spawned chain.
Have the Open risk say the one-time spawn also covers the prime, whose #340 workaround D10 removes.

### [LOW] D2 does not name its liveness test, so Windows parity is unstated and unchecked

**Location:** `decision-record.md` D2, Constraints (fslink/Windows bullet), Acceptance criteria.
**Issue:** D2 says "whose pane is alive" but does not name the test.
The engine already has two: `liveIDSet` (present) and `aliveIDSet` (present and not `pane_dead`).
`--if-absent`'s `classifyIfAbsent` uses the latter, a hidden strand (no pane) never counts, and both backends feed the same `pane_dead` parse.
The Constraints require Windows parity, but the only criterion is a real-tmux test, which runs on POSIX only.
Matching itself is sound: `--name`'s rule (`agentname.Resolve`) expands `orch` to this worktree's full name, so the flag cannot match another worktree's strand or a numbered `orch-2`, and orch adds its strand under that exact name (`orchcli` `orchStrandName`).
That part is not a defect.
**Fix:** State in D2 that "live" is the set `--if-absent` uses (a visible strand whose pane is present and not dead; a hidden strand never counts), computed in the shared engine with no backend-specific code.
Add a fixture-level unit criterion for the skip decision that runs on every platform.

### [LOW] The tracked-tasks.json branch is unspecified for settings.json and the exclude

**Location:** `decision-record.md` D4, D5; Acceptance criteria (`Spawn` overwrites…).
**Issue:** D4 says a tracked `tasks.json` "is left alone with one `logger.Warn`, as in `SpawnDriven`".
`SpawnDriven`'s tracked branch, however, writes neither file and skips `ExcludeAnchoredDir`.
D4 also keeps `settings.json`'s write-when-absent rule.
The record never says whether `Spawn` with a tracked `tasks.json` still writes an absent `settings.json` or the exclude line.
It also never says the tracked check precedes every write, which D4's overwrite depends on, since the check reads the same file the write replaces.
**Fix:** State in D4 that `Spawn` runs `PathTracked` before any write.
When the file is tracked, `Spawn` writes neither file and no exclude line, logs one Warn and still launches, the prime's workspace file included.
Add the absent-`settings.json` tracked case to the criterion.

### [LOW] D6's board exemption is bounded by ownership, not by writer and location

**Location:** `decision-record.md` D6; Scope Out (board bullet).
**Issue:** "The board repo, which lyx owns, is exempt" bounds the exemption by a property any future lyx-owned writer can claim.
It also misdescribes its target: `boardengine`'s `ensureLockfilesIgnored` writes the `.gitignore` of `_board`, the weft's `main` worktree, not a separate repo.
D6's own scope ("a warp or prime worktree") already leaves weft worktrees out, so the exemption as worded is either redundant or wider than intended.
"Generalizes" the junction bullet is also inexact: that bullet binds both sides, while D6 binds warp only.
D7 is consistent with the Constraints' "each caller keeps the loader it adopts today"; that part is not a defect.
**Fix:** Bind D6 to warp worktrees, the prime included.
Name the one weft-side `.gitignore` writer that stays: `boardengine`'s lock and manifest patterns in `_board`.
Say D6 extends the junction bullet's rule to every warp-side writer.
Align the Scope Out bullet.

### [LOW] D8 has no acceptance criterion

**Location:** `decision-record.md` Acceptance criteria.
**Issue:** Every other Decision has a checkable criterion; D8's rewrite of the reconcile-adoption wording, and its kept absent-file hint under `Load`, have none.
**Fix:** Add a criterion: no template comment, Go string or doc under `internal/`, `docs/` or `contracts/` says an existing hub must run `lyx config reconcile` to adopt a new key, and `Load`'s absent-file error still names `lyx config reconcile`.

## Directive verdicts with no finding

- D2's Rationale mechanism narrative stays in `decision-record.md`: its first sentence carries the constraint that the collapse is a stacking effect, not window size, which bounds what the plan writer changes, and three sentences of rationale is what a Decision is for.
- The `window-size latest` Auto-mode assumption stays: it is an autonomous pick against half of the board's stated fix, which is exactly what that section records.
- D7's wording stays consistent with the Constraints line on loader adoption.
