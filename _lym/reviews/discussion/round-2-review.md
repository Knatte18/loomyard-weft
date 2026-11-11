---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 (known directories paragraph; tokencount and report bullet; id-only liveness bullet)
    summary: D6's known-directory set needs the machine-layer accounts, the env module and every hub worktree record, but the readers that use it are built with no config Dirs or are told only a path or an id, and an engine may not derive hub geometry
    fix: In D6, add a paragraph stating how the known-directory set reaches each reader. Recommended - shuttleengines.RecordReaders takes the hub's config Dirs (Board and Anchor) so the claude engine's reading capability can load claude.accounts and the env module, and the hub's worktree-record directories reach the engine as a told list of absolute paths its caller resolves (the engine derives no geometry, PATTERN-cwd-resolution Told geometry); a reader run outside a hub keeps the process's CLAUDE_CONFIG_DIR and the default. Name the same input for the id-only ProcessLiveness reader. Add an acceptance criterion that a reader built for a hub finds a session under a listed account's directory with the worktree and its record gone.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D6 third bullet; acceptance criteria D6 lym report bullets; Scope bullet 5
    summary: lym report reads no transcripts - it prices landed runs from the usage records committed in each run's tree - so D6 names the wrong reader, and the session-end usage recorder that does read transcripts has no rule or criterion
    fix: In D6, replace lym report in the third bullet with the session-end usage recorder (shedadapters' transcript fill, which calls LocateSessions with the session's worktree) beside tools/tokencount, and state that lym report reads the usage records that recorder writes. In Scope bullet 5 change lym report to the session-end usage records. Rewrite the two D6 lym report acceptance criteria as - the session-end usage record of a session run on a listed non-default account carries its transcript's figures, and tools/tokencount counts a landed run's sessions on such an account with the worktree and record gone.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 resume bullets; D6 resume check
    summary: Shuttle runs the resume check before Prepare, so a resume with no record whose transcript sits in a listed non-default directory is refused before D2's search can find and record it, and a resume over a gone recorded directory gets the check's generic refusal, not D2's
    fix: In D2 and D6, state that the resume check itself resolves the directory by D2's rules before it stats the transcript - with no record it searches every known directory for the session's transcript, and with a recorded directory that is gone it refuses naming the directory and the way forward - and that Prepare then records the directory the check found. Add an acceptance criterion - a resume with no record whose transcript is in a listed non-default account's directory passes the check and records that directory.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D2 fresh-launch bullet ("the session ids its shuttle run records and reed strands hold"); D3 step 1; D7
    summary: The fresh-launch search rests on a list of the worktree's resumable session ids that Prepare is never handed, and the named sources miss the orch's own state record, which holds the session id the orch resumes
    fix: In D2, name the sources of the worktree's resumable session ids as the shuttle run records, the reed strands and the orch's state record under the worktree, and state how they reach the claude engine - recommended, the caller passes them to Prepare and OpenLaunch as told data, so the engine reads no other module's records. Add an acceptance criterion - a fresh launch in a prime whose orch state record names a session whose transcript is in the default directory records the default directory and makes no pick.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 first bullet; D2 no-record fallback; D6 no-record fallback and rationale
    summary: An exported account CLAUDE_CONFIG_DIR is inherited by every lym process run from that pane, and reed spawns its tmux server from os.Environ minus a scrub that strips only CLAUDECODE and CLAUDE_CODE_*, so a server started from such a process runs every later no-export pane under that account
    fix: In D4, add a bullet - the claude engine's scrub (ScrubbedEnv) also names CLAUDE_CONFIG_DIR, so reed's server spawn never carries a pane's account into the server's environment, and the process value the readers fall back to is the operator's own. Add an acceptance criterion - a tmux server spawned from a process whose environment holds CLAUDE_CONFIG_DIR carries none of it.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 seeding paragraph ("Counts for names no longer listed are kept and ignored") and rationale
    summary: A re-added account keeps its stale count and a raised weight halves an account's count over weight, so either takes a run of picks in a burst, against D3's claims of no catch-up and even spacing
    fix: In D3, replace the seeding paragraph with - at each pick, an account whose count over weight is below the lowest count over weight among the other listed accounts that have a count is raised to that lowest value times its weight before the comparison, so a new, re-added or reweighted account takes its share from the next pick and never catches up; with no listed account holding a count, every count starts at 0. Add an acceptance criterion - an account re-added after other accounts took picks, and an account whose weight was raised, never takes two picks in a row while another eligible account's share is behind.
  - id: F7
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 merge rules and the renamed-account rationale
    summary: A name reused for another directory keeps the old account's usage file, and the merge drops the new account's windows whose resets_at is earlier than the stored one, so the new account is judged on the old account's figures for up to seven days
    fix: In D5, add to the usage file the cleaned config directory the engine resolved at launch, passed to the verb beside the name, and state that an incoming window whose directory differs from the stored file's replaces the whole file rather than merging. Add an acceptance criterion - after a name moves to another directory, the first status line input for it replaces the stored windows whatever their resets.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D1 ("told by the layer provenance configengine reports for each unit")
    summary: configengine's provenance names only the highest layer holding a key, so a tracked accounts list beneath a machine-layer list is masked and never refused by the mechanism D1 names
    fix: In D1, replace "told by the layer provenance configengine reports for each unit" with "read from each tracked layer's own document in the module's stacked layers, so a tracked value is refused even when the machine layer also holds the key". Add to the D1 tracked-layer acceptance criterion - also when the machine file lists accounts.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 ("A record whose directory exists is kept")
    summary: A kept record whose directory still exists but whose login was removed puts every fresh session of that worktree at Claude Code's login prompt, and the record states no bound or way forward for that case
    fix: In D2, after "Removing an account from the list stops new picks of it", add - a record whose directory exists but is logged out is kept too; its sessions stop at Claude Code's login prompt, and the way forward is to log the directory in again or remove the record so the next fresh session searches and picks.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md acceptance criteria, D2 resume-with-no-record bullets
    summary: The D2 criteria that a resume with no record records the default or the env module's directory omit that accounts are listed, which contradicts D2's first bullet and the D1 criterion that with an empty list a resume writes no record
    fix: In the two D2 acceptance criteria "a resume launch in a worktree with no record, its transcript in the default directory" and "a resume launch in a worktree with no record, its transcript under the env module's CLAUDE_CONFIG_DIR", insert "with claude.accounts listing an account" after "no record".
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 invalid-list sentence; D6 known directories
    summary: D1 fails a launch on an invalid list, but D6 does not say what the readers whose known directories include the listed accounts' do when the list fails to load
    fix: In D6, after the known-directories sentence, add - when claude.accounts fails to load, a reader logs the refusal and reads the remaining known directories, so a bad list never hides the sessions already recorded on the other directories.
  - id: F12
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 ("The command runs a lym verb with that account's name as its argument")
    summary: The verb must find the hub's usage directory and the account list, but D5 does not say how, and the status line command's working directory is not one lym's cwd resolution can rely on
    fix: In D5, state that the rendered status line command names the lym binary through selfexec.LymBinary and runs from the session's project directory, as the creel hook commands do with CLAUDE_PROJECT_DIR, so the verb resolves its hub from the worktree's anchor; no path argument is added, so the verb's input stays the name and stdin.
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 seeding sentence
    summary: The seed is undefined when no listed account has a count, as on the hub's first pick or when the file holds only dropped names
    fix: Fold into F6's rewrite of the seeding paragraph - with no listed account holding a count, every count starts at 0.
---

### [BLOCKING] D6's known directories cannot reach the readers that use them

**Location:** `_lym/discussion/decision-record.md` D6; acceptance criteria D6.
**Issue:** `shuttleengines.RecordReaders` builds every engine with `configengine.Dirs{}` (`internal/shuttleengines/shuttleengines.go:222-228`), and it serves `tools/tokencount` (`tools/tokencount/main.go:341`) and the session-end usage recorder (`internal/loomcli/wiring.go:640`). The machine layer stacks only when `Dirs` carries both `Anchor` and `Board` (`internal/configengine/doc.go`), so these readers load no `claude.accounts` and no env module. "The directories of the hub's worktree records" needs the hub's worktree list, which an engine may not derive (`PATTERN-cwd-resolution`, Told geometry), and `LocateSessions` is told one path and `ProcessLiveness` one id. A landed run's sessions on a listed non-default account are then never found, which is the cost and usage purpose of Scope bullet 5. Raised by bound-breaker, existing-paths and walkthrough, and found by the handler.

### [MEDIUM] lym report reads no transcripts

**Location:** `_lym/discussion/decision-record.md` D6 third bullet; acceptance criteria.
**Issue:** `internal/reportengine/cost.go` prices landed runs from "the usage records it keeps in its tree", and `reportengine` calls no `LocateSessions`. The transcript read happens at session end in `internal/shedadapters/sessionusage.go`, which falls back to shuttle's counts when no transcript is found. The two `lym report` criteria test a path that never reads a config directory.

### [MEDIUM] The resume check runs before Prepare

**Location:** D2 resume bullets; D6.
**Issue:** `internal/shuttleengine/run.go:581` runs `checkResume` and `run.go:642` runs `engine.Prepare`. With no record, D6's check looks only at the env module's, the process's or the default directory, so a transcript on a listed account is refused before D2's search and record write. D2's own way forward, removing a record, leads to this for any worktree on a non-default account. Raised by interactions and walkthrough.

### [MEDIUM] The search's session ids have no stated source or path into the engine

**Location:** D2 fresh-launch bullet; D3 step 1; D7.
**Issue:** `Claude.Prepare(runDir, spec, cfg)` gets one session id in `Spec`. The record names shuttle run records and reed strands but not how they reach the engine. The orch resumes from its own state record (`internal/orchengine/state.go:88`), which the named sources miss, so a prime whose orch strand left reed's state can pick away from the orch's transcript. Walkthrough and interactions findings merged.

### [MEDIUM] An account's CLAUDE_CONFIG_DIR leaks into the tmux server

**Location:** D4; D2 and D6 no-record fallbacks.
**Issue:** reed's server spawn takes `os.Environ()` through the engine scrub (`internal/reedengine/lifecycle.go:456`), and the claude scrub names only `CLAUDECODE` and `CLAUDE_CODE_` (`internal/shuttleengine/claudeengine/scrub.go:9`). A server spawned by a lym process inside a pane exported to account B carries B, and every later pane with no export, such as with an empty list and no record, runs under B. That breaks the "stands as today" criterion and D6's claim that the fallback matches what Claude Code would use in a pane.

### [MEDIUM] The counter bursts for a re-added or reweighted account

**Location:** D3 seeding paragraph.
**Issue:** Counts for names no longer listed are kept, so a re-added account resumes at its stale low count and takes every pick until it catches up. Raising a weight halves the account's ratio and causes the same burst. Both contradict D3's own rationale.

### [LOW] A reused name's usage file blocks the new account's figures

**Location:** D5 merge rules.
**Issue:** The stored file keeps the old account's later `resets_at`, and the merge drops incoming windows with an earlier reset, so the new directory is judged on the old account's figures until that reset. D5 claims that a reused name files the figures under the entry that names the directory now. This is downgraded from the lens's MEDIUM, since it needs an operator rename and lasts at most one window.

### [LOW] D1 names a provenance mechanism that cannot see a masked tracked value

**Location:** D1.
**Issue:** `Stack.Origins` and `Provenance` (`internal/configengine/layers.go:589-686`) name the highest layer only. Raised by walkthrough and bound-breaker, and found by the handler.

### [LOW] A logged-out directory that still exists keeps its worktrees

**Location:** D2.
**Issue:** D2 bounds a gone directory and D3 an expired login at pick time, but not a kept record whose directory is logged out.

### [LOW] D2 resume criteria omit the listed-accounts condition

**Location:** acceptance criteria.
**Issue:** Read literally, the criteria contradict D2's first bullet, which writes no record with an empty list.

### [LOW] Readers on an invalid list

**Location:** D1; D6.
**Issue:** D1 fails a launch on an invalid list, but nothing says how a reader that needs the list behaves.

### [LOW] How the status line verb finds its hub

**Location:** D5.
**Issue:** The verb writes under the hub's `.lym` and checks the list, and the creel hooks `cd` to `CLAUDE_PROJECT_DIR` before they run `lym` (`internal/shuttleengine/claudeengine/settings.go`). D5 names neither the cwd nor how the binary is named.

### [NIT] Seed undefined with no counts

**Location:** D3.
**Issue:** "The lowest count over weight among the listed accounts that have a count" has no value on the first pick.

## Rejected

- None. Every fork finding was kept, merged as a duplicate or, in one case, downgraded (F7).
