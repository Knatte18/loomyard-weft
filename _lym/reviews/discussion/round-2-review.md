---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:239 (D11 Bound)
    summary: An old owner whose dobby is down when the owner moves, after a reboot or a beam stop, never archives its unpushed tips, since beam start refuses a foreign owner and a starting dobby exits on its first probe.
    fix: In D11, decide an entry point that archives without a live dobby and state it in the decision, for example running the same fabric archive call from the foreign-owner refusal of `lym beam start` (and so from `lym s`) before it refuses, under the slug's teardown lock, with a matching acceptance criterion; if the record keeps the dobby as the only entry point, replace the Bound line "A machine that is off when the owner moves archives nothing until its dobby runs again" with the true bound, that a machine whose dobby is not running archives nothing, and name the hand way forward.
  - id: F2
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:98 (D3) and :128-133 (D5)
    summary: Dropping the settled-runs step leaves an adopted beam's own-scope owner pause entry, written by a beam start's owner pause or a hibernate wake, standing with nothing that lifts, converts or names it, since D5 converts owner entries only for moved runs and released tasks.
    fix: In D5, add the adopted subjects' own pair slugs (each adopted beam, and no slug for root) to the slugs whose existing `OwnerCaller` entry of their own scope the `pause` step turns into an `AdoptCaller` entry in place, so `lym tender resume <beam>` lifts it and the run report and `lym beam status` name it; add the case to the D5 acceptance criterion on conversion.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:53-55 (D2)
    summary: D2 builds the end envelope from the outcomes of every done step, earlier processes' steps included, yet its list of what the record holds stores no step outcomes, so a resumed worker cannot report the steps an earlier process ran.
    fix: In D2's field list at line 53, change "the steps done with their end times" to "the steps done with their end times and each one's outcome, the part of the end envelope that step contributes".
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:287 (Acceptance criteria, D5)
    summary: The D5 criterion says each adopted run and each released task gets an adopt pause entry, dropping D5's two exceptions, a slug holding another caller's entry and a front task.
    fix: Rewrite the bullet at line 287 to read that each adopted run, live here or not, and each released task that is not a front task gets an `adopt` pause entry, except a slug already holding an entry of another caller, which keeps it and gets none; an `owner` entry is turned into one in place, keeping its pause time.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:281 (Acceptance criteria, D2)
    summary: The criterion that a discard racing a starting worker leaves its record standing holds only when the worker took adopt.lock first; when the discard holds it, D2 has the worker exit writing nothing.
    fix: Replace "a discard that races a starting worker leaves its record standing" at line 281 with "a discard while a worker holds `adopt.lock` is refused and leaves its record standing, and a worker that meets a discard holding the lock exits writing nothing, its caller exiting non-zero naming the re-run".
---

### [MEDIUM] D11's old owner archives nothing once its dobby is down

**Location:** `_lym/discussion/decision-record.md:239`, D11 Bound.

**Issue:** The only archive trigger is the old owner's running dobby.
A machine rebooted, or whose beam was stopped, while the owner moved has no dobby, and none can start for that subject.
`lym beam start` refuses a foreign owner before any dobby starts, and a dobby that does start reads the owner on its first probe (`internal/dobbyengine/loop.go:494-500`).
`lym s` there runs `adopt --here`, which adopts the subject back.
So the Bound's "until its dobby runs again" names an event that never comes.
The tips stay local and `lym adopt --archives` on the new owner lists nothing, so D11 misses its own purpose in the common case of a laptop shut down after a session.

### [LOW] An adopted beam's own owner pause outlives the adopt

**Location:** `_lym/discussion/decision-record.md:98` (D3) and `:128-133` (D5).

**Issue:** Today's settled list starts from the adopted beam slugs, and `liftOwnerPauses` lifts each one's own-scope `OwnerCaller` entry (`internal/adoptcli/adopt.go:398-407`, `:1005-1020`).
A beam start's `pauseOwnedRuns` writes such an entry for every owned-elsewhere entry, beam projects included (`internal/orchcli/start.go:395-411`).
D3 drops that lift, but D5 converts owner entries only for moved runs and released tasks, so a beam with no unfinished batten record keeps an `owner` entry.
Nothing lifts or names it after the adopt: the dobby's owner lift reaches only runs it drives.
The orch reads only the hub-scope entry (`internal/orchengine/pause.go:81`), so it is not held.
The beam pair's other sessions stay paused under an entry labelled as another machine's.

### [LOW] The record keeps no step outcomes for the envelope it promises

**Location:** `_lym/discussion/decision-record.md:53-55`, D2.

**Issue:** Line 55 builds the end envelope from "the outcomes of every step the record lists as done, earlier processes' steps included".
Lines 53-54 list what the record holds, and no step outcome is on that list.
A worker resuming at `deploy` therefore holds nothing to report the reconcile, pairs, pause or owner steps with.
Today's envelope carries all of them (`internal/adoptcli/cli.go` help text).
Round 1's F11 fix added the envelope to the record but not the step outcomes that build it.

### [LOW] The D5 acceptance criterion drops D5's exceptions

**Location:** `_lym/discussion/decision-record.md:287`.

**Issue:** The criterion asserts an `adopt` entry for every adopted run and every released task.
D5 gives none to a slug already holding another caller's entry (line 133), and none to a front task (line 130).
A test written to the criterion would assert entries D5 forbids.

### [LOW] The discard race criterion covers one interleaving only

**Location:** `_lym/discussion/decision-record.md:281`.

**Issue:** When the worker takes `adopt.lock` first, the discard is refused and the record stands.
When the discard holds the lock, D2 line 58 has the worker exit and write nothing, so no record of that worker stands.
The criterion states the first case as if it covered both.

## Focus verdicts

- Resume paths in D1, D2 and D10: settled.
  `--here` reads the record first, takes up a worktree whose subject the unfinished record lists before `check`, and resumes at the first step not done past `owner`.
  The caller exits non-zero unless the record ends `done`, and `runArgvs` stops at the first non-zero argv (`internal/shortcutcli/shortcut.go:218`).
  So no `beam start` runs before the adopt ends; acceptance criteria D10 at lines 296-297 pin it.
- D4's covered worktrees: settled.
  A beam with no owner of its own reads its owner through `hubgeom.ParentOf`, and D4 covers it.
  Its watcher and dobby neither stop, exit, archive nor start a pass until the subject's owner write; criterion line 285 pins it.
- Race and staleness bounds in D1 and D2: the bounds hold against the code.
  The anchor's `EndSparing` spares only push and sync (`internal/fabriccli/anchor.go:468`), and no other path ends a hub-owner process.
  The start-time rule keeps a lost-lock caller off an older `done` record.
  The direct worker call is bounded by validation, subject match and `adopt.lock`.
  A resume at or before `owner` re-reconciles; a third machine's later adopt is then an explicit takeover.
  A resume past `owner` reads no origin, which D2 and D4 state.
  The one wording gap is F5.
- D10 and D11 against existing code and `PATTERN-owner-machine`: the claims hold.
  `prepare` only warns on an unreachable origin (`internal/orchcli/start.go:446-447`).
  `ArchiveTagSlug`, `TagRefs(ArchiveTagPrefix)`, `remoteArchiveTags` and the remote-warp cleanup all match `archive/` and never `archive-owner/`.
  The owner rule's push is a pair's commit and push, and the owned-elsewhere teardown precedent stands.
  The gap is F1, the archive's trigger.
- D12's listing: settled.
  It lists only `archive-owner/` tags.
  It reads each repository's tags from that repository's origin and compares them with that repository's branch of the slug.

## Rejected

- bound-breaker F3, a direct `lym adopt worker` call from an orch session starts orchs: the orch skip is a convenience carried on the request, not a guard. The hidden verb's bound in D1 is about which subjects it may touch. An orch that runs adopt is already up, and `BeamUp` leaves a live orch alone. An orch start for another subject starts no work, since D5 pauses every adopted run.
