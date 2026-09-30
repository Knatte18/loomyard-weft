---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:34
    summary: The once-per-run gate is not defined precisely enough to implement - it leaves open whether it means no marker present or first spawn, what clears the marker between runs of one slug, and which spawn sites it covers; the resumed-run risk and AC1 each assume a different reading
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:50
    summary: The .git/info/exclude write is under-specified for a shared file - no serialisation with fabricengine's lock and atomic replace, no pattern anchoring that covers an AnchorRel subpath, and it is worded as a per-worktree file
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:45
    summary: Always overwriting tasks.json contradicts never modifying a tracked file when the child repo tracks .vscode/tasks.json
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md:66
    summary: The record does not state what happens to the once-per-run guarantee when Run-Shed's machine-local scratch is lost (machine change, cleared .lyx)
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md:56
    summary: The premise that the folderOpen reed attach reaches the child's own session is never stated as a requirement - VS Code must open the same directory the child's bootstrap ran in
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md:94
    summary: The folder-and-color Decision has no acceptance criterion, while others map one-to-one
  - id: F7
    severity: NIT
    location: _lyx/discussion/decision-record.md:34
    summary: The spawn-confirmation marker is called -spawn-confirmed but its real suffix is -spawned
  - id: F8
    severity: NIT
    location: _lyx/discussion/decision-record.md:30
    summary: The closure Decision carries an illustrative signature and a rationale clause that re-argues the rejected split-row alternative already in support-log.md
---

### [MEDIUM] Once-per-run gate is ambiguous and has no lifetime rule

**Location:** `_lyx/discussion/decision-record.md:34` (Decision), `:83` (Open risk), `:88`–`:89` (Acceptance criteria), `:12` (Scope).

**Issue:** The Decision gates the open on "a marker under `Run-Shed`'s own scratch directory", which reads as "open after any successful spawn when no once-marker exists".
The Scope ("right after its first successful spawn") and the Open risk ("a run resumed from an older build past its first spawn has no once-marker and no further first spawn, so no window opens") instead read it as "first spawn of the run".
Under the marker reading that risk is false: a resumed run with no marker opens on its next successful spawn (an approval re-spawn in `callAwaiting`, or a running-without-confirmation re-spawn).
Also, `innerrun.go` has two spawn sites (`Call`'s read-before-spawn branch and `callAwaiting`), but AC1 ("before the post-spawn status read decides the row's outcome") describes only the first; `callAwaiting` has no post-spawn status read.
Most important, the record gives the marker no lifetime.
`Run-Shed`'s scratch directory (`shedrun.ScratchDir(prime, slug)`, under `.lyx`) outlives a run of the slug; `innerrun.go:129` already clears a leftover done-seen marker because "a marker from an earlier run of the same slug" can survive.
A once-marker left by an earlier run of the same slug (re-run after `doneSlugRefusal`'s run-directory delete) would silently suppress the open on every later run.

**Fix:** State the gate as "marker absent" at both spawn sites.
State that the marker is cleared when `Call` finds no child status file (a fresh child), before that spawn.
Rewrite the resumed-run risk to match, and make AC1/AC2 cover both spawn sites and a leftover marker from an earlier run.

### [MEDIUM] `.git/info/exclude` write is under-specified for a shared file

**Location:** `_lyx/discussion/decision-record.md:50` (Decision), `:82` (Open risk), `:94` (Acceptance criterion), `:106` (Note).

**Issue:** The Decision says "the task worktree's `.git/info/exclude`", but the task worktree is a linked worktree and git resolves `info/exclude` to the common gitdir (`fabricengine/gitexclude.go` header), so one file serves every worktree of the repo.
The Open risk admits this but treats it as harmless because the entry is idempotent.
Idempotence of the entry does not make the write safe.
`fabricengine` serialises every read-modify-write of that file under the `exclude.lyx.lock` flock and replaces it atomically, because an unsynchronised write loses concurrent updates and can truncate the operator's own patterns.
A driven-path writer outside that lock races `lyx fabric` verbs and concurrent batten runs.
The helper (`mutateGitExclude`) is unexported, so the Note's "check whether a helper there fits" does not steer the plan writer to the requirement.
Separately, `.vscode/` is written at `<AnchorRel>/.vscode/`.
`fabricengine`'s junction entries are root-anchored (leading `/`), so reusing that pattern style yields `/.vscode/`, which misses a subpath-anchored hub, and the AC's "`.vscode/` is ignored" would fail there.

**Fix:** Add a Constraint: resolve the file with `git rev-parse --git-path info/exclude`, write it under `fabricengine`'s exclude lock with an atomic replace (by exporting or wrapping the existing helper, never a second locking scheme), and use a pattern that matches `.vscode/` at the anchor subpath.
Reword the Decision to "the repository's shared `info/exclude`".
Extend the AC to a subpath-anchored hub.

### [MEDIUM] Overwriting `tasks.json` contradicts "never modifies a tracked file"

**Location:** `_lyx/discussion/decision-record.md:45`–`:51`, `:77`.

**Issue:** The driven path "always writes `tasks.json`, overwriting an existing one" and "never modifies a tracked file".
Batten's child can be any hub's repo, and some repos commit `.vscode/tasks.json`.
There, the overwrite dirties the tracked tree the child's loom run commits in, which is the failure the `.gitignore` Decision exists to avoid.
The Auto-mode assumption covers only hand edits in a batten-created pair, not a tracked file.

**Fix:** State the tracked case: when `.vscode/tasks.json` is tracked in the task worktree, the driven path leaves it untouched and logs a warning, and the open still happens.
Add a matching AC and record the pick in the support log.

### [LOW] No stated behaviour when scratch is lost

**Location:** `_lyx/discussion/decision-record.md:66` (Constraint), `:34` (Decision).

**Issue:** The marker lives in machine-local, ephemeral `.lyx` scratch.
When that scratch is lost (the run resumed on another machine, or `.lyx` cleared), the spawn-confirmation marker goes with it.
`Call` then re-spawns the running child, and with no once-marker the open fires again.
The record neither accepts nor forbids this, so the plan writer cannot tell whether it is intended.

**Fix:** State it in the Decision: a lost marker re-opens the window on the next successful spawn, which is the intended behaviour on a new machine.

### [LOW] Attach-targets-the-child's-session premise is implicit

**Location:** `_lyx/discussion/decision-record.md:42`, `:56`.

**Issue:** The design works only if `lyx reed attach`, run on `folderOpen` in the opened folder, resolves the same reed session the child's `loom start --no-attach` brought up.
It does, because `Spawn` runs the bootstrap with `cmd.Dir = taskLocation.AnchorPath()` and reed's geometry comes from that location, and batten opens `<hub>/<slug>/<AnchorRel>`.
The record states the folder only as a Decision about color and the prime branch, never as the requirement that keeps the attach on the child's session.
The strand-is-up premise is stated adequately as a Decision.

**Fix:** Add to the folder Decision that the opened folder must be the directory the child's bootstrap ran in, so the attach resolves the child's own session.

### [LOW] Folder/color Decision has no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md:94`–`:96`.

**Issue:** Every other Decision has a matching criterion.
The folder Decision (`<hub>/<slug>/<AnchorRel>`, color picked as `ide spawn` picks it) has none, and it carries the session-targeting requirement from the previous finding.

**Fix:** Add a criterion: the stubbed `ideengine.CodeLauncher` receives `<hub>/<slug>/<AnchorRel>`.

### [NIT] Wrong marker name

**Location:** `_lyx/discussion/decision-record.md:34`.

**Issue:** The record names an existing `-spawn-confirmed` marker; `innerrun.go` uses the suffix `-spawned` (`SpawnConfirmedFile`).

**Fix:** Name the marker by its declarer, `SpawnConfirmedFile`, rather than a suffix.

### [NIT] Deliberation and a signature in a Decision

**Location:** `_lyx/discussion/decision-record.md:30`–`:32`.

**Issue:** "(e.g. `OpenIDE func(ctx) error`)" is an illustrative signature, a fact class the writer stencil leaves to Plan time.
The rationale clause "a separate row would require splitting `Run-Shed`'s spawn out of it, duplicating … and changing the durable row set" re-argues the split-row alternative already in support-log.md's Rejected alternatives.
Its one requirement, that recipe row names and order stay unchanged, is already a Scope item and a Constraint, so moving it loses nothing.

**Fix:** Drop the signature and the re-argued clause; keep the positive rationale.

## Focus verdicts not covered by a finding

- Acceptance-criteria testability: the criteria are observable as written.
  The warning-line criterion is testable (`innerrun_awaiting_test.go` already captures the logger), and "before the post-spawn status read" is testable by stub call order.
  The gaps are the coverage gaps in F1, F3 and F6.
- Placement from support-log.md into decision-record.md: no move needed.
  Interview item 6's requirement is already Scope "Out", item 7's is already the tracked-file Decision (its `mill`-block detail is specific to this repo, not to batten's child repos), and the `lyx board get` exit-status note is an operator fact, not a plan-writer requirement.
