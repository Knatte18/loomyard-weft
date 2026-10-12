---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:119
    summary: D10 keeps the newest handoff by file name, which after an aborted cycle is a partial PendingHandoff file, so it deletes State.LastHandoff, the one file a restart resumes from, and its bound is false
    fix: In D10, replace "removes every file under the orch's handoffs directory except the newest, newest by the timestamp in its name" with "removes every file under the orch's handoffs directory except the one the persisted State.LastHandoff names; with LastHandoff empty or naming a missing file it removes nothing and says so". Change D10's Bound to "the file State.LastHandoff names always survives, so a restart still resumes from it". Change the doc.go sentence D10 prescribes to state that the handoff LastHandoff names is kept. In Acceptance criteria, replace "keeps the newest handoff by name and removes the rest" with "keeps the handoff State.LastHandoff names and removes the rest, including a newer partial handoff an aborted cycle left; with no LastHandoff it removes nothing".
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:109
    summary: D9 empties the slug's scratch directory except run.lock, which deletes status.json.lock and last-commit.lock, contradicting its own bound, and it holds only the run lock while lyx batten run reads and seeds status.json under the status lock without the run lock
    fix: In D9, replace the scratch bullet with "Then, still holding the run lock and also holding the slug's status lock (shedrun.StatusLock) from before the records run directory is removed until the commit is made, it removes the slug's .lyx run scratch files except every lock file (run.lock, status.json.lock, last-commit.lock and any other *.lock), and releases both locks; lock files and their directory stay, so no lock is deleted while held or recreated beside a held one". Apply the same lock-file exception to the orphan bullet. Keep the Bound's "never touches ... a lock file" as is, now true. Add to Acceptance criteria the row "a lyx batten run <slug> started while batten cleanup --apply holds the slug neither re-seeds status.json before the deletion commit nor locks a new status-lock inode".
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:108
    summary: D9's tracked-but-deleted candidate has no done check, since the on-disk status.json it would read is gone, so any slug whose records run directory vanished on disk gets its deletion committed and pushed
    fix: In D9, replace the tracked-but-deleted bullet with "A slug whose records run directory is deleted on disk but still tracked is a candidate only when the status.json in the records HEAD tree decodes to done and the run lock can be taken; otherwise it is skipped and reported, so an apply interrupted between removal and commit is finished by the next run and no other on-disk deletion is committed". Add the Acceptance row "a tracked-but-deleted run directory whose committed status is not done is skipped and reported, its deletion left uncommitted".
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:31
    summary: D1 requires mode 0700 on every platform, but Go reports Windows directory permissions as 0777 (or 0555), so the root check always fails on Windows and every caller errors there
    fix: In D1, change "with mode 0700 and, off Windows, owned by the current user" to "and, off Windows, with mode 0700 and owned by the current user; on Windows only the real-directory (not a link or reparse point) check applies". In Acceptance criteria, change the root row to "A root that is a link is an error on every platform; off Windows, a root with another owner or not 0700 is too; each error names the path and nothing is created in the bare system temp directory".
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:55
    summary: D4 removes the run directory after m.Run on Windows but leaves open whether tmuxkit.Main now redirects TMPDIR, TMP and TEMP there on Windows, and what a failed removal does; with the redirect, treadleengine's gate_lingering_test leaves a ping child whose cwd is os.TempDir(), which blocks the removal on Windows
    fix: In D4's tmuxkit bullet, add "On Windows it sets TMPDIR, TMP and TEMP to the run directory as setEnv does today. A removal that fails, such as one a lingering child's cwd blocks on Windows, is reported on stderr and does not change the exit code; the next start's sweep removes the directory once its lock is free." Add the Acceptance row "a run directory whose removal a lingering child blocks is reported, leaves the exit code unchanged and is removed by the next start's sweep".
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:5
    summary: The Goal promises no temp garbage, but D4 covers only test binaries with a TestMain; 44 test packages use t.TempDir with no TestMain, so their trees stay in the bare system temp directory after a timeout or kill, and D6 allows t.TempDir everywhere
    fix: Add a D4 bullet "A test package with no TestMain gets one calling tmuxkit.Main, so its t.TempDir trees live in the process run directory; cmd/lyx's tmux-isolation scan fails a test package without a TestMain that calls tmuxkit.Main, except its allowlist entries", and add the matching Acceptance row "a test package with no TestMain of its own is refused by the scan". If the author instead keeps those packages out, add "t.TempDir trees of a test package with no TestMain, after a timeout or kill" to Scope Out and narrow the Goal's first clause to the binaries D4 names.
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:71
    summary: D6's allowlist is an escape hatch with no stated bound on what an entry lets through, and it contradicts the support log's Q3 (empty allowlist) while Open risks already expects a reedengine entry
    fix: In D6, replace the allowlist sentence with "A site that cannot move is admitted by an allowlist entry naming one file and the creation call in it, with a reason naming the obstacle; an admitted site still removes what it creates on normal exit, and what it lets through is that one site's directory left in the bare system temp directory after a kill. The allowlist lives in the scan test, so every entry is a reviewed code change." In support-log.md's Question ledger, change Q3's auto-pick to "ban bare system-temp creation outside the temp package; allowlist entries only for sites that cannot move, each naming its obstacle (D6)".
  - id: F8
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:91
    summary: D7 bounds lyx cleanup against each segment's function, not its verb, so a refusal that lives only in a verb's CLI layer (D9's prime-only and lock checks, D10's idle-phase check) is skipped by the in-process call
    fix: In D7's Bound, replace "removes nothing a segment's own function would not remove" with "removes nothing a segment's own verb would refuse to remove; every refusal and skip of batten cleanup, board prune, orch cleanup and the .scratch segment lives in the module function both the verb and lyx cleanup call, and the verb adds only flag parsing and output". Add the Acceptance row "lyx cleanup's orch segment outside the idle phase removes nothing, as lyx orch cleanup does".
  - id: F9
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:128
    summary: D11's board-citation guard does not say what a board read failure does, so an unreadable board can degrade to no citations and let the segment remove cited evidence
    fix: In D11, after the citation sentence, add "A board that cannot be read fails the segment and removes nothing." Extend D11's Bound to "only direct children of the prime's .scratch/, and only with the board read". Add to the .scratch Acceptance row "a board read failure fails the segment with nothing removed".
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:127
    summary: D11 calls the cleanup module the one declarer of the .scratch literal, but tools/sandbox/report.go already joins .scratch for its reports and internal/testkit/scankit lists it, and no decision moves either
    fix: In D11, delete ", and is the one declarer of the `.scratch` literal" from the first sentence.
  - id: F11
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:41
    summary: D2 leaves the order of the age and lock checks open, and a lock attempt on an entry with no lock file creates one, so a sweep that tries the lock first writes inside a fresh live run directory and can make the owner's own lock attempt fail, against D2's own bound
    fix: In D2, replace the removal sentence with "Within the creation grace an entry is skipped untouched. Past it, an entry with no lock file is removed, and an entry with one is removed when its lock can be taken (owner dead); the sweep never creates a lock file. Age is the directory's modification time against an injectable clock." In D1, add "The owner takes its lock with a blocking acquire, so a sweeper's momentary hold delays it rather than failing it."
  - id: F12
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:48
    summary: D3 adopts a root derived from an inherited TMPDIR without saying it passes D1's trust checks, which leaves the trust-boundary rule undecided for the child path
    fix: In D3, after "takes that run directory's root as its own", add "after the same real-directory, mode and owner checks D1 applies, failing the same way when they fail".
  - id: F13
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:110
    summary: D9's aged-orphan rule rests on the records run directory being absent, but does not say a stat error other than not-exist counts as present, so an unreadable or unlinked _lyx would empty the scratch of paused or blocked runs, which hold no run lock
    fix: In D9's orphan bullet, add "A records run directory counts as absent only on a not-exist error; any other stat error skips the slug and is reported."
---

### [BLOCKING] Orch cleanup keeps the newest handoff by name, not the one a restart resumes from

**Location:** `_lyx/discussion/decision-record.md:119` (D10), and its Bound at line 122.

**Issue:** The watcher names a new handoff with `NewHandoffPath(w.paths, now)` and records it as `PendingHandoff` when a cycle starts (`internal/orchengine/watcher.go:721-727`).
An aborted cycle runs `toIdle`, which clears `PendingHandoff` in state (`watcher.go:381-384`) but leaves any file the session already wrote on disk.
`LastHandoff` moves only when a cycle passes its gate (`watcher.go:845`, `:1160`), and a launch resumes from `State.LastHandoff` alone, never a pending one, "since an aborted cycle may have left it partial" (`internal/orchengine/launch.go:42-56`, `doc.go:37-38`).
So after an aborted cycle the newest file by name is the partial one.
D10 keeps it and deletes the file `LastHandoff` names, and the next launch falls through to the start stencil instead of resuming.
The phase guard does not help, since `toIdle` already returned the phase to idle.
The Bound's "the newest handoff always survives, so a restart still finds it" is false.

### [BLOCKING] Batten cleanup deletes lock files and races a concurrent batten run

**Location:** `_lyx/discussion/decision-record.md:109` (D9 scratch bullet), against the Bound at line 114.

**Issue:** The prime's scratch directory for a done slug holds `run.lock`, `status.json.lock` and `last-commit.lock`, plus markers (seen in the prime's `.lyx/shed/agent-naming/`).
`shedrun.StatusLock` and `shedrun.RunLock` are separate files under `ScratchDir` (`internal/shedrun/paths.go`).
D9 keeps only the run lock file, so it deletes `status.json.lock` and `last-commit.lock`, which contradicts its own "never touches ... a lock file".
D9 also holds only the run lock, but `battenPreRun` reads the status under the status lock and seeds a fresh `status.json` when it is absent (`internal/battencli/arm.go:436-497`), all before the run-lock probe in the pre-step.
So a `lyx batten run <slug>` started mid-cleanup can find no status, seed a new one, and have that file swept into or out of the cleanup's records commit.
Meanwhile, the deleted status lock lets a later reader lock a new inode beside a held one.
This is the hazard the support log's own rejected alternative names for the run lock.

### [MEDIUM] Tracked-but-deleted candidates skip the done check

**Location:** `_lyx/discussion/decision-record.md:108`.

**Issue:** The done-only guard (line 105) reads the persisted status, but this candidate class exists because the records run directory, `status.json` included, is gone from disk.
As written, any slug whose run directory vanished on disk becomes a candidate, and its deletion is committed and pushed through the records path.
That covers a slug deleted by hand or by mistake while paused or blocked.
`shedrun` already offers tree-relative status paths for a git tree read (`paths.go` header), so the committed status can serve as the proof.
The Decision introduces a routing edge around the done guard with no stated bound, which the rubric's attack-surface rule flags.

### [MEDIUM] The 0700 root check cannot pass on Windows

**Location:** `_lyx/discussion/decision-record.md:31`, and its Acceptance row at line 156.

**Issue:** D1 puts "off Windows" on the ownership check only, so mode 0700 is required everywhere.
On Windows, `os.Stat` reports directory permissions as 0777, or 0555 for a read-only attribute, never 0700.
Every Windows caller would get the error, and by D1 it may not fall back, so every Windows test binary and `lyx gate test` would fail.
Constraints and D4 both commit to Windows support, so D1 contradicts them.

### [MEDIUM] Windows redirect and removal failure are undecided in D4

**Location:** `_lyx/discussion/decision-record.md:55`.

**Issue:** Today `tmuxkit.Main` returns `m.Run()` on Windows before `setEnv` (`internal/testkit/tmuxkit/tmuxkit.go:69-71`).
D4 says the run directory is "the `TMPDIR`/`TMUX_TMPDIR` target" and is removed on every platform, but not whether Windows now gets the redirect.
Acceptance (line 159) requires nothing outside the root on Windows, which needs the redirect for `t.TempDir` trees.
With the redirect, `internal/treadleengine/gate_lingering_test.go:39-64` starts a `ping` that outlives the test with `os.TempDir()` as its cwd, by design, and Windows refuses to remove a directory that a live process uses as its cwd.
D4 does not say whether that removal failure fails the binary.

### [MEDIUM] Test packages without a TestMain still leak on kill

**Location:** `_lyx/discussion/decision-record.md:5` (Goal), against D4 (lines 52-60) and D6 line 72.

**Issue:** Go's `t.TempDir` creates under `os.TempDir()`, and a timed-out or killed binary skips its cleanup.
44 test packages use `t.TempDir()` and declare no `TestMain` (for example `internal/lock`, `internal/orchengine`, `internal/configengine`), so `TMPDIR` is never redirected for them.
D4's bullets only cover binaries with a `TestMain`, and D6 allows `t.TempDir` without condition.
Those trees stay in the bare system temp directory, which the Goal and Acceptance line 160 ("A killed test binary's run directory is removed by the next start's sweep") do not account for.
This is filed once as a method gap, not per package.

### [MEDIUM] The scan's allowlist is an unbounded escape hatch and disagrees with Q3

**Location:** `_lyx/discussion/decision-record.md:71`; `_lyx/discussion/support-log.md:65`; Open risks at decision-record line 150.

**Issue:** D6 admits allowlist entries with only a reason requirement.
It does not say what an entry lets through, how narrow it is (a file, a call or a package), or whether an admitted site must still clean up.
The rubric's attack-surface rule requires that bound.
Q3 records the pick as "empty allowlist", D6 provides one, and Open risks expects a `reedengine` entry for `socketfile_test.go:117` and `:177`, so the three disagree.
The existing scan `cmd/lyx/tmuxisolation_test.go:23` shows the house form: an entry is admitted only for a named obstacle.

### [MEDIUM] In-process segments can skip their verbs' refusals

**Location:** `_lyx/discussion/decision-record.md:83-91`.

**Issue:** D7 calls each owning module's function in-process and bounds the result by what "a segment's own function" would remove.
Several of the guards that make the segments safe are stated as verb behaviour: D9's "prime only" and its skips, and D10's "does nothing, and says so, unless the persisted cycle phase is idle".
If any of them lands in the CLI layer, `lyx cleanup` bypasses it, and the bound as worded does not forbid that.
This answers the focus question on whether a segment can remove what its own verb would refuse: as written, yes.

### [MEDIUM] Board read failure is unbounded in the .scratch guard

**Location:** `_lyx/discussion/decision-record.md:128`.

**Issue:** The citation check is the guard that keeps evidence open tasks point at.
D11 does not say what happens when the board cannot be read.
A fail-open reading treats every old child as uncited and removes it.
The attack-surface rule asks for the bound on a guard that can be bypassed.

### [LOW] The .scratch sole-declarer claim is false

**Location:** `_lyx/discussion/decision-record.md:127`.

**Issue:** `tools/sandbox/report.go:129` joins `".scratch"`, and `internal/testkit/scankit/scankit.go:38` lists it.
No Decision moves either, and no `PATTERN.md` entry backs a sole-declarer rule, so the claim is neither true nor enforced.

### [LOW] The sweep's check order is open, and a lock attempt can write inside a live directory

**Location:** `_lyx/discussion/decision-record.md:41-43`.

**Issue:** `lock.TryAcquireWriteLock` uses `flock.New(path).TryLock()` (`internal/lock/lock.go:37-47`), which creates the lock file when it is missing.
If the sweep tries the lock before checking age, it creates and holds a lock file inside a run directory whose owner has not locked yet.
That contradicts D2's "never reads inside a live run directory".
An owner using a try-lock would then fail its own lock.
D2's wording, "lock file can be taken ... or it has no lock file, and its age is past a creation grace", does not fix the order.

### [LOW] A root adopted through D3 has no stated trust check

**Location:** `_lyx/discussion/decision-record.md:48`.

**Issue:** D1 validates "the root", and D3 derives the root from an inherited `TMPDIR`, which is environment input.
Neither says the adopted root goes through D1's checks.
On the focus question of whether the trust-boundary rule is decided consistently across D1, D4 and Constraints, this is the one gap.
The Windows unlock-before-remove ordering agrees across D1 line 33 and Constraints line 137.

### [LOW] The orphan rule fails open on a stat error

**Location:** `_lyx/discussion/decision-record.md:110`.

**Issue:** Paused, blocked and failed batten runs hold no run lock, and their scratch carries markers that matter on resume: park, resume report, done-seen, halt-warned and revived (`internal/shedrun/paths.go`, `internal/battenshed/innerrun.go:382-405`).
The only thing separating them from an orphan is that their records run directory exists.
The rule does not say a stat error other than not-exist must count as present.

## Focus verdicts

- D2 and D3, deleting a live process's files: with a held lock the sweep never removes the entry.
  A directory made just before the grace boundary is past the grace only after its owner has had 10 minutes to lock, so it is safe once the check order in F11 is fixed.
  A child beside its parent sees the parent's held lock and leaves it alone.
  Processes that outlive a killed parent and use its directory as `TMPDIR` lose it to the sweep, which is the leak D4 targets, not a defect.
- D6 against Q3: unbounded and inconsistent, see F7.
- D9 and D11, every deletion path: tracked-but-deleted (F3), aged orphan (F13), `.scratch` links (bounded by the same age rule, not a defect) and the board-citation check (F9).
  A running or still-referenced task's state is reachable through the lock-file deletion and the re-seed race (F2).
- D7 and D8: dry run and `--apply` share one candidate function, which is sound.
  In-process calls can skip a verb's refusal (F8).
  Continue-on-failure lets no segment remove more than its own rules allow, since no segment depends on another's result.
  D8's premise holds: `fabricengine.Remove`, which `pairteardown` drives, commits pending records and archives the weft tip (`remove.go:2`), while `prune.go` and `cleanup.go` take no lock shared with it.
  `Cleanup` also archives the tip before deleting a branch (`cleanup.go:124`), so the archive alone does not separate them, but the board-claim settle and pending-records commit do.
- D1, D4 and Constraints: Windows removal ordering is consistent.
  The trust rule has the F4 and F12 gaps.
  Every requirement in the support log's Interview is present in the record: the gitconfig and killed-binary leaks, lock liveness, the scan shape, the segments, `.lyx` disposability and the `.scratch` owner.
