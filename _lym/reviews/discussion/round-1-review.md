---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3 (claude-file row) and Open risks line 3
    summary: >-
      A rendered CLAUDE.local.md makes Claude Code stop reading a repository's own AGENTS.md, hiding the team's instructions, and the record leaves Claude's CLAUDE.local.md behaviour as an open risk the docs already answer
    fix: >-
      In D3, record the fact with its evidence (Claude Code memory docs, code.claude.com/docs/en/memory, sections AGENTS.md and Choose which instruction files load; behaviour from v2.1.277, checked on v2.1.316):
      Claude Code loads CLAUDE.md and CLAUDE.local.md from the working directory and every parent, and under its default Project instructions value claude-md-or-agents-md a CLAUDE.local.md in the working directory or above counts as a CLAUDE.md, so Claude then reads no AGENTS.md.
      Then decide how the claude-file target keeps a repository's AGENTS.md reaching Claude when the worktree holds no CLAUDE.md or .claude/CLAUDE.md, the way the codex-file row already does for Codex:
      for example the claude-file render puts the repository's root AGENTS.md (or an @AGENTS.md import line) before the row's entries when no CLAUDE.md exists, and AGENTS.md and CLAUDE.md join that row's re-render paths;
      or claudeengine's settings file sets pluginConfigs cc-plugin-agents-md@builtin instructionFiles to claude-md-and-agents-md, stating that it covers lym-launched sessions only.
      Add a D3 acceptance case for a worktree with a root AGENTS.md and no CLAUDE.md.
      Delete Open risks line 3 (Claude Code's support for CLAUDE.local.md is assumed...), since the fact is now recorded in D3.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D9 second bullet
    summary: >-
      Setting runproc.HubOwner from the lymcwd constant makes internal/runproc import internal/lymcwd, which PATTERN-leaf-packages forbids, and the record declares no change to that entry
    fix: >-
      Settle in D9 how runproc.HubOwner takes the name without breaking PATTERN-leaf-packages (runproc imports only the standard library, internal/proc, internal/state and internal/logger).
      Either add a bullet under PATTERN changes, PATTERN-leaf-packages changes, adding internal/lymcwd to runproc's closed import set (lymcwd imports neither runproc nor anything that does, so no cycle), and add a Constraints line naming it;
      or keep runproc on the standard library and have D9 name another way, such as the one literal living in a leaf package both lymcwd and runproc may import, with the PATTERN-cwd-resolution and PATTERN-leaf-packages changes that needs declared under PATTERN changes.
      Extend D9's first bullet's import argument to cover the runproc edge, and keep the D9 acceptance criterion that runproc.HubOwner derives from the one declaration.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3 last render bullet, D5, Open risks line 4
    summary: >-
      D3's refusal checks the warp_exclude config list, not the seeded info/exclude line, so lym heddle render or a take-in render in a worktree made before this lands writes an untracked file into warp
    fix: >-
      In D3, make the guard hold for worktrees fabric has not re-wired since landing.
      Either the render has fabric seed the warp_exclude lines first through an exported fabricengine call (fabric owns the exclude mutation and its lock, so heddle runs no git), or the render refuses a target whose anchored line is absent from the warp repository's info/exclude, naming lym fabric reconcile as the way forward.
      Reword Open risks line 4 to name the step that seeds the lines before the render, and add a D3 acceptance case for a render in a worktree whose info/exclude lacks the lines.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 first render bullet
    summary: >-
      The in-place write rests on two renders of one worktree writing the same text, which fails when heddle.json or AGENTS.md changes between their reads, so a detached take-in render overlapping a hand render can leave a corrupt file
    fix: >-
      In D3, bound concurrent renders of one worktree: state that a render of a worktree holds a per-worktree lock under the worktree's never-tracked .lym tree (taken through internal/lock, its path named by internal/lymdirs) while it reads its inputs and writes the files, so two renders serialize and the last one wins whole.
      Replace the sentence two renders of one worktree write the same text with that bound, and add a D3 acceptance case that two renders with different inputs leave one render's whole text.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 seam-failure sentence
    summary: >-
      The downgraded guard's bound, a warning reported in the verb's result, reaches no reader when the verb is the detached sync D6 itself names, so a failed take-in render leaves a stale AGENTS.override.md with only a log line
    fix: >-
      In D6, state where a seam failure in a detached process lands and what bounds it: for example the detached sync writes a friction note or a creel notice to the hub naming the worktree and lym heddle render as the way forward, or D6 states plainly that a detached failure is log-only and that the stale file stays until the next take-in of a re-render path or a hand render.
      Add the matching D6 acceptance case.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Scope Out bullet 6 against D9 third bullet and D2
    summary: >-
      Scope keeps fabricengine.BoardDir and the write lock out of the task while D9 requires the Bolt write lock and everything else in Bolt beside the board to take its place from BoltDirName, and fabric locates the Bolt checkout and the lock through BoardDir today
    fix: >-
      In D9's third bullet, state which constructor gives the Bolt checkout's path (for example a fabricengine.BoltDir (new) built from BoltDirName, with BoardDir built from BoardDirName) and that fabric's uses of BoardDir meaning the Bolt checkout, BoardWriteLockPath included, move to it.
      Narrow Scope's Out bullet 6 to board names in fabric outside the replay, the ended set and the owner read that are not renamed, such as the write lock's file name board.lock, and drop fabricengine.BoardDir from it.
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md PATTERN changes, PATTERN-heddle-instructions
    summary: >-
      The added PATTERN text says only internal/heddleengine writes a passage into a launched session's system prompt and that heddle.json is written only by lym heddle verbs, which D4 (claudeengine writes the append file), D8 (the Bolt replayer rewrites heddle.json) and shuttle's existing prompt and instruction files all break
    fix: >-
      Reword the PATTERN-heddle-instructions adds text so it binds only heddle's content and its real writers, for example: Rendering heddle entries into a worktree file or a session's system prompt: only from heddle.json in the Bolt checkout, through internal/heddleengine (new); that file written only by internal/heddleengine, through lym heddle verbs and its Bolt replayer; fabric names no heddle file.
      Keep the rule text under the PATTERN rule-text cap that lym loom validate-discussion applies.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 adopted-resume sentence and Auto-mode assumptions last line; support-log.md Rejected alternatives
    summary: >-
      D4 lets an adopted resume read heddle afresh, so its passages can change mid-run, while the board entry's rule that an agent's rules do not shift mid-run and the support log's rejection of re-reading on resume are left unreconciled with it
    fix: >-
      In D4, state the adopted resume's re-read as a bounded exception to the no-mid-run-shift rule: an adopted session's run directory holds no earlier passages, so it takes heddle as it stands at adoption.
      In support-log.md's Rejected alternatives, narrow the Re-reading heddle on resume item to a resume through the launch's own resume line.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D7 set-aside bullet and closing sentence
    summary: >-
      A non-ledger replayer's set-aside tag set-aside/<name>/... has the same shape as a run's, so a replayer named heddle collides with a run slug heddle, and setAsideTag's empty-slug fallback still names board
    fix: >-
      In D7, give a non-ledger replayer's set-aside tag a shape no run slug can take, for example set-aside/replayer/<name>/<short sha>, or reserve replayer names as fabric reserves hub geometry names against slugs;
      and state that setAsideTag's empty-slug fallback no longer names the board.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D7 last bullet
    summary: >-
      D7 states the failure for a replay and an ended-set read with no run ledger registered, but not for the origin owner read, which today reads board.json itself
    fix: >-
      In D7's last bullet, add the origin owner read to the reads that fail as ErrBoardReplayMissing fails today, naming the registration as the way forward, when no run ledger is registered.
  - id: F11
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 stale-render bullet against D3 exclude refusal and D5
    summary: >-
      A path taken off warp_exclude makes D3 refuse its target, so heddle's stale marked render of it is never removed
    fix: >-
      In D3, state that removing a stale marked render is never refused by the exclude-list check, which guards writes only.
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks line 1; support-log.md Question ledger Q11
    summary: >-
      The literal's home is a choice the writer settles from the import graph D9 already cites, not a question only the parent can settle, since the board entry's wording cannot be met while fabricengine imports lymcwd
    fix: >-
      Delete Open risks line 1 and mark Q11 in support-log.md's Question ledger resolved as an auto-pick citing D9's import argument, once F2's fix settles runproc's side of it.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 fourth bullet and D9 acceptance criterion
    summary: >-
      Every test taking the name from the constant contradicts TestEnforcement_GeometryLiterals, which must spell the _board token it polices
    fix: >-
      In D9's fourth bullet and its acceptance criterion, exempt the geometry literal test's token table: the _board literal appears in one Go declaration and in that test's policed-token list.
  - id: F14
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md Acceptance criteria D7
    summary: >-
      The criterion that no done status is left in fabric's replay or ended-set code is false for the run status done the ended-set narrowing reads
    fix: >-
      In the D7 acceptance criterion, change done status to board status done.
---

### [BLOCKING] A rendered CLAUDE.local.md hides the repository's AGENTS.md from Claude

**Location:** `_lym/discussion/decision-record.md` D3, Open risks line 3

**Issue:** Claude Code's memory docs say that under the default `claude-md-or-agents-md` value, a `CLAUDE.local.md` in the working directory or above counts as a `CLAUDE.md`, and Claude then reads no `AGENTS.md`. The docs warn that adding one stops Claude from reading `AGENTS.md`. The installed Claude Code is 2.1.316, and the behaviour applies from 2.1.277. Heddle's `claude-file` render therefore hides a team's `AGENTS.md` from every Claude session in the worktree, which is the outcome the board entry rules out and D3 already prevents for Codex. Open risks also leaves Claude's `CLAUDE.local.md` support to be found out later, though the same docs answer it now.

### [BLOCKING] D9 makes runproc import lymcwd, against PATTERN-leaf-packages

**Location:** `_lym/discussion/decision-record.md` D9

**Issue:** `pattern/PATTERN-leaf-packages.md` limits `internal/runproc` to the standard library, `internal/proc`, `internal/state` and `internal/logger`, and its current imports match. D9 sets `runproc.HubOwner` from the `lymcwd` constant and declares no PATTERN change. The import argument in D9 covers only the `fabricengine` edge.

### [MEDIUM] The exclude guard reads config, not the seeded line

**Location:** `_lym/discussion/decision-record.md` D3, D5, Open risks

**Issue:** D5 seeds lines only on Add, AddLocal, CloneHub and reconcile (`seedGitExclude` is reached only from the wiring path at `internal/fabricengine/junction.go:138`). The record's own remedy for existing worktrees, `lym heddle render`, and a take-in render both write the file with no exclude line. The file then shows as untracked in warp.

### [MEDIUM] Concurrent in-place renders are unbounded

**Location:** `_lym/discussion/decision-record.md` D3

**Issue:** D6 renders from `heddle.json` as it stands, and a detached sync's take-in can overlap a hand render. Two truncate-and-write calls with different texts can leave the shorter text over the longer one's tail. The claim that two renders write the same text holds only when their inputs match.

### [MEDIUM] A detached seam failure reaches no one

**Location:** `_lym/discussion/decision-record.md` D6

**Issue:** The bound on the downgraded guard is a warning reported in the verb's result. The detached sync D6 names has no reader for that result, so a failed re-render leaves a stale `AGENTS.override.md` with only a log line.

### [MEDIUM] Scope keeps BoardDir out while D9 moves the Bolt paths off it

**Location:** `_lym/discussion/decision-record.md` Scope, D9, D2

**Issue:** `BoardWriteLockPath` joins `BoardDir(hub)` with `board.lock` (`internal/fabricengine/junctionnames.go:92`). Fabric also locates the Bolt checkout through `BoardDir` (`hubpull.go:74`, `hubworkspace.go:304`). D9 requires the lock and every Bolt-beside-board item to take its place from `BoltDirName`, while Scope's Out bullet keeps `BoardDir` and the lock out of the task.

### [MEDIUM] The added PATTERN text is broken by the design itself

**Location:** `_lym/discussion/decision-record.md` PATTERN changes

**Issue:** `PATTERN-heddle-instructions` says only `internal/heddleengine` writes a passage into a launched session's system prompt. D4 has `claudeengine`'s `Prepare` write those passages into the append file, and `claudeengine` already writes lym's own sentences there (`internal/shuttleengine/claudeengine/settings.go:444`). The text also says only `lym heddle` verbs write `heddle.json`, while D8's Bolt replayer rewrites it during a fabric replay. As a declared entry it would be judged by its listed text, so the implementation D4 and D8 prescribe would violate it.

### [LOW] The adopted-resume re-read is not reconciled with the no-mid-run-shift rule

**Location:** `_lym/discussion/decision-record.md` D4; `_lym/discussion/support-log.md` Rejected alternatives

**Issue:** D4 accepts that an adopted session reads heddle afresh. The support log rejects re-reading on resume because the prompt would change under the session. Neither text bounds the exception.

### [LOW] Set-aside tags for a replayer share the run-slug shape

**Location:** `_lym/discussion/decision-record.md` D7

**Issue:** `setAsideTag` builds `set-aside/<slug>/<short sha>` and falls back to `board` for an empty slug (`internal/fabricengine/replay.go:483-491`). A replayer named `heddle` collides with a run slug `heddle`.

### [LOW] The owner read with no run ledger is unstated

**Location:** `_lym/discussion/decision-record.md` D7

**Issue:** `readOriginOwners` reads `board.json` itself today (`internal/fabricengine/originreconcile.go:823`). D7 names the failure with no run ledger for the replay and the ended set, but not for the owner read.

### [LOW] A stale render is never removed once its path leaves the list

**Location:** `_lym/discussion/decision-record.md` D3

**Issue:** The off-list refusal also blocks the stale-render removal, so heddle's file stays in the worktree.

### [LOW] Open risk Q11 is a writer's choice, not a parent question

**Location:** `_lym/discussion/decision-record.md` Open risks line 1; `_lym/discussion/support-log.md` Q11

**Issue:** `fabricengine` imports `lymcwd`, so `lymcwd` cannot take the name from `fabricengine`. The entry's wording cannot be met as written, and D9 already cites the import graph. This is no outdated entry, conflict with another run, or doubt about what the task is for, so the writer settles it.

### [LOW] Every test from the constant contradicts the geometry literal test

**Location:** `_lym/discussion/decision-record.md` D9

**Issue:** `TestEnforcement_GeometryLiterals` lists `_board` as a policed token (`internal/lymcwd/enforcement_test.go:209`), so it cannot take the name from the constant.

### [NIT] The D7 acceptance criterion names the wrong done

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria

**Issue:** `internal/fabricengine/replay.go:289` reads a run status `done`, which is not a board status and is not what the criterion means.

## Rejected

- walkthrough 4 (the made event names no payload): the event's parameters are an interface shape, which the writer's spec leaves to Plan. The record already constrains heddle to receive absolute paths (Constraints, PATTERN-cwd-resolution).
- bound-breaker 6 (a committed render carries the marker): with the lines seeded, a render becomes tracked only through a forced add. F3 closes the unseeded path that made it plausible.
- interactions' pair line `runproc may import lymcwd: held`: contradicted by `pattern/PATTERN-leaf-packages.md`. Kept as F2 from the walkthrough and bound-breaker lenses.

## Focus verdicts

- "Relocation pass over `decision-record.md`: Constraints, Auto-mode assumptions and Open risks against Decisions": I made the pass and raised no relocation finding. Auto-mode assumptions restate their Decisions by design, since that section lists the autonomous picks. The Constraints lines add the PATTERN binding each Decision leaves out. Open risks lines 1 and 3 are handled under F12 and F1, which drop them on other grounds.
- "D7, ended-run set-aside narrowed to the ledger part": I reached it and found no defect in the bound. D7 states what now gets through, a non-ledger replayer's change in a commit that also carries an ended run's ledger change, and what bounds it: the change goes through that replayer's own by-entry merge and refusals, and the original is still tagged. The tag-naming defect next to it is F9.
