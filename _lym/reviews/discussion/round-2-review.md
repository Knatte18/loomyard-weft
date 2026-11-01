---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 (first render bullet, "It writes the file in place, which a session reads only when it starts")
    summary: False premise — Claude Code re-reads project-root instruction files from disk after compaction, so an in-place render can be read half-written by a live session and changes a live session's instructions at its next compaction
    fix: In D3, replace the in-place bullet with an atomic write — the render writes a temporary file whose path fabric's exclude list covers (add its pattern to the template default of D5's warp_exclude, or put it under the pair's never-tracked .lym tree on the same filesystem) and renames it onto the target, under the per-worktree render lock; state, citing code.claude.com/docs/en/memory section "Instructions seem lost after /compact" (checked on v2.1.316), that a live session re-reads the root file at its next compaction, so a take-in or hand render reaches live sessions then. In the support log, correct Turn 4's bound-breaker 5 line and drop or reverse the rejected alternative "An atomic write by a temporary file renamed into place". Add to D3's acceptance a case that a render replaces the file atomically and leaves no untracked file in warp.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 render bullets and D1 ("An absent file holds no entry")
    summary: A render reads heddle.json under no Bolt lock, while git moves of the Bolt checkout (Bolt pull fast-forward, replay, reconcileRecordsBranch's ffWorktree) replace the file non-atomically, so a concurrent render can see it absent, read no entries and silently remove a live worktree's marked renders
    fix: In D3, state that a render reads heddle.json while holding the Bolt write lock (fabricengine.BoardWriteLockFile in the Bolt checkout, which Bolt's pull, replay and reconcile take), taken only around the read and released before the render writes, so lock order is the render lock then the Bolt write lock and no fabric caller holding the Bolt lock fires the seam. Add to D3's acceptance a case that a render concurrent with a Bolt fast-forward over a heddle.json change renders either the old or the new entries, never none.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 ("In a detached process, such as the board's Bolt sync, the failure is log-only") and the D6 acceptance criterion
    summary: The board's Bolt sync only commits and pushes the Bolt checkout and never moves a warp code branch, and D6 itself says a Bolt move fires nothing, so the named example is wrong and its acceptance case cannot be built
    fix: In D6, replace the Bolt sync example with a process that moves a warp code branch with no foreground caller to report to, such as a ReconcileOrigin call from battencli or hibernatecli, or state the rule as any process whose verb has no foreground result; in the D6 acceptance criterion, replace "a failing seam in a detached Bolt sync" with that process's case.
  - id: F4
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3 (claude-file row)
    summary: Claude Code also reads .claude/AGENTS.md and a CLAUDE.local.md counts against it, so a render in a repository whose only AGENTS.md is .claude/AGENTS.md hides the team's instructions from Claude, which D3's rationale rules out
    fix: In D3's claude-file row, write the import line for whichever of AGENTS.md or .claude/AGENTS.md the worktree's root holds (an @.claude/AGENTS.md import for the latter), and add .claude/AGENTS.md to that row's re-render paths, citing the memory docs section "When Claude Code reads AGENTS.md"; extend the D3 acceptance case with the .claude/AGENTS.md repository.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D9 first bullet and Constraints (PATTERN-leaf-packages)
    summary: D9 says PATTERN-leaf-packages's background admits runproc importing lymcwd, but today's background lists only the standard library, proc, state and logger for runproc, and no line of the record names that background edit as a change to make
    fix: In D9's first bullet, replace "PATTERN-leaf-packages's background admits it, as Constraints states" with "PATTERN-leaf-packages's background gains internal/lymcwd in internal/runproc's import set, for the one Bolt directory name"; in Constraints, name the same background edit.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D9 (fourth bullet) and the D9 acceptance criterion
    summary: TestEnforcement_GeometryLiterals spells the _board literal not only in its policed-token list but as its owner-row key and in its planted fixture sources, so the D9 exception and acceptance criterion as written cannot both hold
    fix: In D9's fourth bullet and the D9 acceptance criterion, widen the exception from the policed-token list to TestEnforcement_GeometryLiterals as a whole, its policed-token list, owner-row key and planted fixture sources.
  - id: F7
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D2 (render verb)
    summary: The new render verb writes files into, and has fabric seed the exclude file of, whatever worktree its argument names, and D2 states no bound on that argument
    fix: In D2's render bullet, state that the argument names a warp worktree of the current hub, resolved through lymcwd.NamedWorktree, and that any other path is refused with the way forward naming lym fabric's worktree list; add the refusal to D2's refusal list and acceptance criterion.
---

### [MEDIUM] In-place render rests on a false premise about when Claude reads the file

**Location:** `_lym/discussion/decision-record.md` D3, first render bullet; support log Turn 4 bound-breaker 5 and the rejected alternative "An atomic write by a temporary file renamed into place".

**Issue:** D3 writes `CLAUDE.local.md` and `AGENTS.override.md` in place, "which a session reads only when it starts".
The Claude Code memory docs (code.claude.com/docs/en/memory, "Instructions seem lost after `/compact`") say the project-root file is re-read from disk after `/compact` and re-injected.
`CLAUDE.local.md` "loads alongside `CLAUDE.md` and is treated the same way".
Lym's sessions compact, and the prime orch lives in the prime worktree, where `lym fabric pull` takes in remote commits and D6 re-renders.
A compaction that coincides with the in-place write can re-inject a truncated file.
The per-worktree render lock serializes renders, not Claude's read.
The rejected atomic rename was rejected only because its temporary file would show as untracked.
A temporary path on fabric's exclude list, or under the never-tracked `.lym` tree, removes that objection.
Lenses existing-paths (F1) and bound-breaker (F2) both raised this, and the handler found it independently.

### [MEDIUM] A render can see `heddle.json` absent during a Bolt checkout move

**Location:** `_lym/discussion/decision-record.md` D3 render bullets; D1 "An absent file holds no entry".

**Issue:** D2 serializes heddle's writes with Bolt's pull and replay through the Bolt write lock.
A render takes no Bolt lock when it reads `heddle.json`.
Git updates a checked-out file by unlinking and recreating it, and these moves do so over a `heddle.json` change: Bolt's pull fast-forward (`internal/fabricengine/bolt.go:307`), a replay's commit, and `reconcileRecordsBranch`'s `ffWorktree` (`internal/fabricengine/originreconcilemoves.go:197`).
A "made" or take-in render running in another process meanwhile reads no file.
D1 treats that as no entries, and D3 then removes every marked render with no warning.

### [MEDIUM] D6's detached example and its acceptance case name a process that never fires the seam

**Location:** `_lym/discussion/decision-record.md` D6, "In a detached process, such as the board's Bolt sync, the failure is log-only", and the D6 acceptance criterion.

**Issue:** `boardengine.Sync` runs `commitDirty` and `Bolt.Push` (`internal/boardengine/sync.go:36-58`).
`PushRecorded` retries through `retryBoltPush` on the Bolt side alone (`internal/fabricengine/bolt.go:104-128`).
No warp code branch moves, and D6 itself says a Bolt move fires nothing.
The acceptance case "a failing seam in a detached Bolt sync logs a warning… and leaves the move intact" therefore has no mechanism for the plan to test.
Raised by interactions, walkthrough and bound-breaker; kept at MEDIUM because the acceptance case it carries cannot be met.

### [LOW] The `@AGENTS.md` import misses a repository's `.claude/AGENTS.md`

**Location:** `_lym/discussion/decision-record.md` D3, `claude-file` row.

**Issue:** The memory docs, section "When Claude Code reads AGENTS.md", list "every `AGENTS.md` and `.claude/AGENTS.md` in your working directory and the directories above it" as read at session start.
A `CLAUDE.local.md` counts against that read.
D3 imports only a root `AGENTS.md`, so in a repository whose only AGENTS file is `.claude/AGENTS.md`, the render hides it.
That is the outcome D3's rationale says the import exists to prevent.
The row's re-render paths omit the path as well.

### [LOW] D9 states the leaf-packages background change as already true

**Location:** `_lym/discussion/decision-record.md` D9 first bullet; Constraints, PATTERN-leaf-packages.

**Issue:** `pattern/PATTERN-leaf-packages.md` says `internal/runproc` "Imports only the standard library, `internal/proc`, `internal/state` and `internal/logger`."
D9 says the background "admits" `internal/lymcwd`.
The `## PATTERN changes` bullet repeats the overview text unchanged, so the edit that makes the claim true is named nowhere a plan writer would carry into the Webster card.

### [LOW] The `_board` literal exception is narrower than the test it names

**Location:** `_lym/discussion/decision-record.md` D9 fourth bullet and the D9 acceptance criterion.

**Issue:** `internal/lymcwd/enforcement_test.go` spells `"_board"` in its policed-token list (line 209), as the owner-row map key (line 225), and in planted fixture sources that prove the scan catches it (lines 412-413, with the `_boardroom` near-token case at 462-464).
The criterion admits the literal only in one declaration and the policed-token list.
The fixtures cannot take the constant without losing their meaning after `bolt-dir-rename`.

### [LOW] `lym heddle render <worktree>` has no stated bound on its target

**Location:** `_lym/discussion/decision-record.md` D2, `render [<worktree>]`.

**Issue:** The rubric requires a Decision that introduces a verb to state what it can reach and what bounds it.
`render` writes the target files into the named worktree.
Through `fabricengine.SeedWarpExclude` (new), it also has fabric rewrite that repository's shared `info/exclude`.
D2's refusals cover a call outside a hub but not an argument naming a path outside this hub's warp worktrees.

## Rejected

- existing-paths F2 (D6's take-in list omits `FastForwardPair` and `syncSideBeforeMerge`): D6 states the rule as "wherever fabric moves a checked-out code branch to origin's commits". The paths after it are instances, and completing that enumeration is excluded by the rubric's "Do not flag" list.
- bound-breaker F3: duplicate of F3, kept from interactions at MEDIUM.
- bound-breaker F6 and walkthrough F3: duplicates of F6.
- walkthrough F1: duplicate of F3.
- walkthrough F2 and bound-breaker F4: kept as F4.
- bound-breaker F5: duplicate of F5.
- existing-paths F1: duplicate of F1.
