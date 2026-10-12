---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D10
    summary: D10's idle-phase check and its removals are not atomic against the watcher, so a handoff that becomes LastHandoff mid-cleanup can be removed and a resume after /clear then points at a missing file
    fix: In D10, replace "It does nothing, and says so, unless the persisted cycle phase is idle" with a sentence stating that `lyx orch cleanup` holds the orch state write lock (`Paths.StateLockPath`) from reading the phase and `LastHandoff` through its last removal, so the watcher cannot enter handoff-requested in between; extend the D10 Bound with "a handoff a cycle starts while cleanup runs is never removed"; add an acceptance criterion that a cycle entering handoff-requested during `lyx orch cleanup --apply` loses no handoff.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D9 (orphan bullet)
    summary: D9's orphan rule matches a tracked-but-deleted slug whose committed status is not done, so its scratch is emptied although D9 says non-done slugs are never touched; the orphan's lock is also unnamed
    fix: In D9's orphan bullet, define an orphan as a `.lyx` run scratch directory whose records run directory is absent on disk and absent from the records HEAD tree, whose run lock (`shedrun.RunLock`) can be taken, and whose newest modification is older than one day; add to the tracked-but-deleted bullet that such a slug's scratch is never emptied by the orphan rule; add the matching acceptance criterion that a tracked-but-deleted run directory with a non-done committed status keeps its scratch contents.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D9 (tracked-but-deleted bullet)
    summary: The status-lock span is tied to before the records run directory is removed, which never happens in the resume case, so a concurrent batten run can seed a fresh status between the HEAD-tree check and the commit
    fix: In D9's status-lock sentence, change "from before the records run directory is removed until the commit is made" to "from before it decides the slug is a candidate (the on-disk status read, or the records HEAD tree read for a tracked-but-deleted run directory) until the commit is made".
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D4 (tmux-isolation scan bullet)
    summary: D4 makes the tmux-isolation scan the guard for temp containment but leaves its allowlist unbounded, since it does not say what an entry lets through or what an entry must carry
    fix: In D4's scan bullet, after "except its allowlist entries", add that an entry names one package with a reason naming why it cannot call `tmuxkit.Main`, that such a package's `TestMain` removes the process run directory as the next bullet says, and that what an entry lets through is that package's tmux isolation and its `t.TempDir` trees left in the bare system temp directory after a kill; add a Question ledger entry recording this pick beside Q3.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md Question ledger Q5 and Interview 6
    summary: Q5 and Interview turn 6 still say orch handoffs beyond the newest and a single kept lock file, while D10 keeps the LastHandoff file and removes a newer one, and D9 keeps every lock file
    fix: In support-log.md, change Q5 to read "batten's run scratch contents (every lock file kept) of a cleaned or aged orphan slug, orch handoffs other than the one `State.LastHandoff` names, logs through the existing retention sweep (D9–D11)", and change Interview turn 6's "orch handoffs beyond newest" to "orch handoffs other than `LastHandoff`'s".
---

### [MEDIUM] D10's idle check and removals are not atomic against the watcher

**Location:** `_lyx/discussion/decision-record.md` D10

**Issue:** The focus question has a yes answer for a handoff left by an aborted cycle.
`ChooseStartPrompt` in `internal/orchengine/launch.go` consults `State.LastHandoff` and never `PendingHandoff`, and D10 keeps the file `LastHandoff` names, so a newer partial file goes and the resume source stays.
The gap is a cycle that starts while cleanup runs.
`LoadState` and `SaveState` in `internal/orchengine/state.go` hold `StateLockPath` only for one read or one write, so D10's "persisted cycle phase is idle" is a check whose lock is released before any removal.
The watcher can then persist handoff-requested with a new `PendingHandoff` (`watcher.go` around `NewHandoffPath`), the session writes it, and cleanup removes it as a file `LastHandoff` does not name.
If the watcher has already seen the file, it moves `LastHandoff` to the note and types `/clear`, and the resume points at a missing file: the orch's context is gone.
The window is narrow, but D10's Bound promises that the resume file always survives, and nothing in the design enforces it.
The watcher persists each phase before its side effect, so holding the state write lock across check and removal closes the window without new machinery.
Separately, `ChooseStartPrompt` also takes an operator `--handoff` flag.
A flagged file other than `LastHandoff` is removed too, but the start then fails loudly ("does not exist") rather than resuming from the wrong context, so that is not a finding.

### [MEDIUM] D9's orphan rule overlaps the tracked-but-deleted rule

**Location:** `_lyx/discussion/decision-record.md` D9, orphan bullet

**Issue:** The orphan rule tests only that the records run directory is absent on disk.
A tracked-but-deleted run directory whose committed status is not done is skipped by the previous bullet, but it also passes the orphan test once its scratch is a day old, so its park marker, resume report, batten-watched marker and step records are emptied.
That contradicts D9's own "running, paused, blocked, awaiting and failed are never touched": restoring the directory from HEAD resumes a run that has lost its ephemeral state.
A paused or blocked run holds no run lock, as round 1's F13 established, so the lock test does not protect such a slug.
"Lock free" names no lock, while the scratch directory holds `run.lock`, `status.json.lock`, `last-commit.lock` and `steps/loop.lock` (`internal/shedrun/paths.go`, `internal/battencli/commitstatus.go`).
On the focus's stat question, D9 does answer it: absent only on a not-exist error, otherwise skip and report.

### [LOW] D9's status-lock span does not cover the resume branch

**Location:** `_lyx/discussion/decision-record.md` D9, tracked-but-deleted bullet

**Issue:** On the focus's lock question, no lock file is deleted while held, because D9 keeps every `*.lock` and their directory.
A concurrent `lyx batten run <slug>` cannot slip in on the normal path either: `checkRunStatus` and `battenPreRun` in `internal/battencli/arm.go` take the status lock with a blocking acquire (`internal/lock`'s `AcquireReadLock`/`AcquireWriteLock`), so they wait for the commit and then seed a fresh status, which the acceptance criterion pins.
The resume branch is the exception.
D9 anchors the status lock to "before the records run directory is removed", and in that branch the removal happened in an earlier process.
Without the lock held from the HEAD-tree read, a batten run can see the absent status and seed a fresh `status.json` before cleanup stages its pathspec.
Cleanup's commit then records the fresh seed instead of a deletion.

### [LOW] D4's tmux-isolation allowlist is an unbounded escape hatch

**Location:** `_lyx/discussion/decision-record.md` D4, scan bullet

**Issue:** D4 makes `cmd/lyx`'s tmux-isolation scan fail every test package without a `TestMain` calling `tmuxkit.Main`, "except its allowlist entries".
That makes the existing `allowedNoTmuxMain` list in `cmd/lyx/tmuxisolation_test.go`, empty today, the escape hatch for temp containment as well as tmux isolation.
D6 states what its own allowlist entry lets through and what it must name, but D4 states neither for this one.
The Question ledger also records only D6's allowlist (Q3).

### [LOW] Support log's Q5 and Interview turn 6 disagree with D9 and D10

**Location:** `_lyx/discussion/support-log.md`, Question ledger Q5 and Interview item 6

**Issue:** Q5 and Interview turn 6 still say "orch handoffs beyond the newest", which is the reading D10 dropped.
D10 keeps the file `LastHandoff` names and removes a newer partial one.
Q5 also says "lock file kept", singular, where D9 keeps every lock file.

## Checked and not a finding

- D1/D3 on Windows: D1 exempts Windows from the mode and owner checks and keeps only the real-directory check, so Go's 0777 report is never tested against. D3 applies "the same root checks D1 applies on its platform" to an inherited root.
- D4/D6 coverage: every test package gets a `TestMain` calling `tmuxkit.Main`, so its `t.TempDir` trees live in the run directory. A failed removal on Windows is reported and swept later. D6's allowlist bound matches Q3. Non-creating `os.TempDir()` uses, such as `internal/treadleengine/gate_lingering_test.go`'s lingering-child cwd, are in Scope Out and land in the run directory under D4.
- D7/D11: `boardengine.Board.Prune(apply)` already holds board prune's logic, and `boardcli` adds only flags and output, so D7's bound holds for that segment. D11 fails the segment on an unreadable board.
- D7's reading that batten is not bound by `PATTERN-refusal-way-forward` matches that background, which binds only modules with a `refusal-spec.md` section (webster, shed, loom).
