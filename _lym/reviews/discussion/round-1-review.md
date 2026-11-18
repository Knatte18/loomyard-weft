---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:180
    summary: D7's in-run relaunch rests on a shuttle in-run respawn that does not exist, names no step ending the still-live limited process that D6's live-holder check then refuses, rebuilds none of the launch state, and removes a run's strand outside the stop verb with no declared PATTERN change
    fix: "Rewrite D7's relaunch paragraph as a defined new mechanism inside internal/shuttleengine, in this order. (1) End the limited session's process and its reed strand from inside the run, and say how the run record is settled so no stopped outcome is recorded and Wait keeps its run. (2) Build a fresh resume launch through the engine's Prepare, so the run directory's launched-account record (D4), the settings file with its status line (D5) and reed's stored command and ResumeCmd all name the new account. (3) Only then run the resume check, whose live-holder test must no longer see the old process. Drop the words 'the way shuttle's own respawn replaces a strand', and in Notes for the plan writer replace 'Shuttle's in-run respawn' with the fact that no in-run relaunch exists today. Either route step (1) through shuttle's stop verb with a new non-stopped mode, or add a ## PATTERN changes section with a `PATTERN-shuttle-provider-seam` — changes: bullet whose text admits the in-run strand replacement beside the stop verb. Extend D7's first acceptance criterion: after the relaunch, the launched-account record, the settings' status line name and reed's stored resume command all name the new account, and the resume is not refused for a live holder."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:102
    summary: A pinned worktree with no entry whose pinned account is blocked resolves to no account, so it and every sibling on the pin run under the env value or ~/.claude, possibly the blocked account or an unlisted one, until blocked_until passes
    fix: "In D4, replace 'is skipped in favour of the entry' with: a pinned account with a future blocked_until is skipped in favour of the entry when the worktree has one; with no entry the pin stands. In D7, say a pinned worktree's limit switch writes the entry for that worktree only, and other worktrees on the pin with no entry stay on the pin. Add an acceptance criterion under D4: a pinned worktree with no entry whose pinned account has a future blocked_until still launches under the pin."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:178
    summary: D7 moves a hub-pinned worktree off its pin on a limit, which the board's own pin example rules out (this hub only on private account 1), and leaves a pin the operator set for terms or ownership reasons bypassable by any limit hit
    fix: "In D7, say that only the beams pin switches on a limit, and that a worktree under a hub pin waits the limit out as today. Adjust D4's pin-skip clause to apply only to the beams pin. Change the first Auto-mode assumption to say a beams-pinned worktree switches while its pin is out and a hub-pinned one waits, citing the board's 'this hub only on private account 1'. Change D7's pinned acceptance criterion to a beams-pinned worktree, and add one: a hub-pinned worktree whose account hits a limit keeps its pin and switches nothing."
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:164
    summary: D6's copied-transcript resume rests on an uncited claim that the 400 invalid-signature error compares no account, on a shared plugins precondition that D10 leaves partly optional (settings.json enabledPlugins, .claude.json MCP servers), and states no fallback for a copied resume the API refuses, though the board names starting fresh
    fix: "In D6's Facts, name the source and version for the claim that the 400 invalid-signature error is the prefix check and compares no account, or drop the claim. Say that the tools must match between the two directories: the plugins, the enabledPlugins in settings.json and the MCP servers in .claude.json. In D10, make linking settings.json required rather than 'as wanted', and add aligning the MCP servers. Add the fallback to D6: a resume whose first turn ends on an API error the engine classifies as a refused resume starts the session fresh in the same run, as the board's 'otherwise the session starts fresh' says. Add an acceptance criterion for it. Then reword the Open risks line to name only what the manual check confirms."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:183
    summary: D7 does not order its switch against shuttle's existing retryAPIError on the same usage-limited turn end, which counts the limit toward APIErrorRetries, re-sends a line into the pane and can end the run OutcomeAPIError
    fix: "In D7, state that a usage-limited turn end that the switch handles is not counted or re-sent by retryAPIError, and that the API-error watch is reset when the relaunched strand starts. Replace 'Otherwise the limit is handled as today' with that ordering and the as-today path for no switch. Add an acceptance criterion: a switched run on a spec with APIErrorRetries records no API-error re-send for the limited turn end and does not end OutcomeAPIError."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:184
    summary: The orch watcher that D7 has restart the orch holds only a session reader, creel and paths, and cannot start a session; the resume path with endOwnHolder lives in orchcli's resumeOrch
    fix: "In D7, name how the watcher restarts the orch on a switch. Either it is told a restart capability backed by orchcli's resume path, which ends the orch's own live holder before resuming, or it posts the restart to the orch's starter. Keep the D7 orch acceptance criterion and add to it that the still-live limited orch process is ended before the resume."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:68
    summary: D2's prune of entries whose worktree directory is gone has no input, because the engine is told only the worktree's name and kind and the file path and may not derive the hub's worktrees under PATTERN-cwd-resolution
    fix: "In D2, name the told input for the prune: the caller that runs the pick passes the hub's current worktree names as told geometry, and the pick drops entries absent from it. Add that input to D4's told-geometry sentence and to the PATTERN-cwd-resolution constraint line."
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:86
    summary: D3 judges eligibility by listed windows only, so an account that lists no windows stays eligible with a future blocked_until, against D5, and step 3's latest blocking reset is undefined for an account blocked only by blocked_until
    fix: "In D3 step 2, say a candidate with a future blocked_until is never eligible, whatever windows it lists. In step 3, count blocked_until as one of the candidate's blocking resets. Add an acceptance criterion under D3: an account listing no windows with a future blocked_until is passed over."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:142
    summary: D5's forged-input bound says one account's figures in one window, but one shell can forge both windows of every listed account, one call per name, so every account reads short for up to seven days and forged later resets replace real figures
    fix: "Rewrite D5's Bounds sentence: any shell that can run the verb can set both windows of every listed account, up to each window's length ahead, and can replace stored figures with a later reset. The effect is bounded to the pick falling back to D3's step 3 and to no blocked_until being written."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:126
    summary: OpenLaunch writes no settings file and passes no --settings when the creel flag directory is empty, so an opened session on a listed account gets no status line, against D5's claim that its status line still records figures
    fix: "In D5, say OpenLaunch writes a settings file carrying the status line whenever the launched account is listed, also with no creel flag directory. Otherwise drop 'its status line still records figures' and state that such a session records nothing. Make D5's settings acceptance criterion name an OpenLaunch session with no creel flag directory."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:159
    summary: D6's 'OpenLaunch brings the newest copy when it builds a resume line' copies nothing, since OpenLaunch mints a new session id whose resume line only reed replays later under the same stored account
    fix: Delete the sentence 'OpenLaunch brings the newest copy in the same way when it builds a resume line.' from D6.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:90
    summary: D3's count reset takes the lowest count over weight among unchanged accounts, which includes a retired weight-0 account and divides 0 by 0
    fix: In D3's reset sentence, change 'among the accounts listed then under an unchanged weight' to 'among the accounts of positive weight listed then under an unchanged weight'.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:175
    summary: D7's switch increments a count but does not write the weights and listing, so a switch onto an account added since the last pick has its increment wiped by the next pick's reset
    fix: In D7's second bullet, replace 'writes it as the worktree's entry and increments its count' with 'writes it as the worktree's entry and writes the counter as D3's step 4 does, with the reset counts, the current weights and listing, and its count incremented'.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:43
    summary: Retiring an account with weight 0 moves no worktree off it, because an entry naming a still-listed retired account resolves under D4, while D1 refuses a pin naming one; the record states neither the behaviour nor the asymmetry
    fix: "In D2, add that an entry naming a retired account reads as no entry, so its worktree runs under its pin or as today and D6 carries the transcript. Add to D2's first acceptance criterion: an entry whose account is retired reads as no entry."
  - id: F15
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:41
    summary: Directories are compared expanded and cleaned without resolving links, so a config_dir that links to ~/.claude is exported rather than unset, the .claude.json move D4 warns of, and two entries can name one directory
    fix: In D1's config_dir bullet, say directories are compared after expanding, cleaning and resolving links, and that a directory that does not resolve is refused at load.
  - id: F16
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:188
    summary: D7's bound for lym claude account set says any session in the worktree, but the verb resolves its worktree from the cwd, so any process that reaches the hub can set any worktree's entry by changing directory
    fix: "In D7's Bound sentence, replace 'any session in the worktree, an agent's included, can run it' with 'any process that can run lym from a worktree's directory, an agent's included, can set that worktree's entry'."
  - id: F17
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:253
    summary: The beams-pin Open risk is a choice the writer can make, not a question for the run's parent, since the board's rule that every session of one worktree runs under one account settles it; no Added-after-Discussion entry or Decision-for note bears on it
    fix: "Move the first Open risks bullet to Auto-mode assumptions as: the beams pin covers every session of a beam worktree, reading the board's spreading workers as the task worktrees under the beams (D1, D9). In support-log.md's Question ledger, change Q11 from open to auto-pick with that answer."
  - id: F18
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:71
    summary: Several Decision rationales argue against rejected alternatives, which the compaction rule puts in support-log.md's Rejected alternatives, and the slug-hash alternative appears there not at all
    fix: "Cut from decision-record.md: D2's rationale sentence 'the cancelled design's per-worktree record grew mostly from the transcript search its loss forced (board, \"Keeping it small\")'; D3's clause 'where a slug hash cannot once accounts are skipped'; D6's clause 'and no session-id search through owner records is needed'; and D7's sentence 'a stop through shuttle's stop verb would end the run as stopped'. Each surrounding requirement is already stated in its Decision body. Add a support-log.md Rejected alternatives bullet: a hash of the slug as the pick, which cannot keep the long-run share once accounts are skipped."
  - id: F19
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:299
    summary: D10's review criterion asks for the manual check's outcome in the landing notes, but the board puts both accounts on OSL-1046 while this run's owner machine is neve, and the record names neither the machine nor when the check runs relative to landing
    fix: "In D10, name the machine the check runs on and say whether it runs before landing or after, by the operator. Replace the Review bullet with: 'Review: the task's landing notes state how the second account is wired and the manual check's steps.' That is what the board asks the task to state."
---

### [BLOCKING] D7's in-run relaunch has no mechanism and is refused by D6

**Location:** `_lym/discussion/decision-record.md:180`

**Issue:** D7 relaunches the limited strand "inside the same run as a resume of the same session id, the way shuttle's own respawn replaces a strand".
Shuttle has no in-run respawn: a respawn is a caller's new run after Attach judges a candidate respawn-eligible (`internal/shuttleengine/attach.go:499-501`), and "Attach never relaunches or moves a strand" (`attach.go:413`).
`retryAPIError` re-sends a line and launches nothing (`internal/shuttleengine/apierrorretry.go:1-2`).
A usage limit leaves the Claude process alive in its pane and registered live, and D6's resume check refuses a live holder in any known registry (`internal/shuttleengine/claudeengine/resume.go:87-112`).
D7 names no step that ends that process.
The only path in D4 that writes the launched-account record, the status-line settings and the stored `ResumeCmd` is `Prepare`, so a relaunch that does not rebuild the launch keeps all three on the old account.
The new session's figures and its next `blocked_until` then land on the old account, and D7 switches again in a loop.
`PATTERN-shuttle-provider-seam`'s Shuttle stop section says every removal of a run's strand goes through the stop verb, and the record declares no PATTERN change for a removal outside it.
Merged from walkthrough 1, bound-breaker F2, existing-paths F2 and interactions F1.

### [MEDIUM] A pinned worktree with no entry falls to no account while its pin is out

**Location:** `_lym/discussion/decision-record.md:102`

**Issue:** D3 never picks a pinned worktree, so pinned worktrees have no entry, apart from one that D7's switch gave.
D4 skips a blocked pin "in favour of the entry", so every other worktree on that pin resolves to none and runs under the `env` value or `~/.claude` until `blocked_until` passes.
That directory may be the blocked account itself or an unlisted one.
The acceptance criterion at line 276 covers only a worktree that has an entry.
Merged from interactions F2 and bound-breaker F1.

### [MEDIUM] A hub pin is switched away from on a limit, against the board's pin example

**Location:** `_lym/discussion/decision-record.md:178`

**Issue:** The board gives "this hub only on private account 1" as a pin, and the word "only" makes the hub pin exclusive.
D7 and the first Auto-mode assumption switch every pinned worktree on a limit, citing the board's "waited out only when no account of the machine has headroom".
Under a hub pin, the accounts open to the hub are the pinned one alone, so waiting the limit out satisfies both board lines.
The beams pin carries no such "only": its stated purpose is Remote Control landing in one account.
As written, any limit hit takes a hub-pinned worktree onto an account the operator excluded, and D4 states no bound for that escape from the pin.

### [MEDIUM] D6's copied-transcript resume: uncited claim, optional precondition, no fallback

**Location:** `_lym/discussion/decision-record.md:164`

**Issue:** Focus directive 1 asks whether D6 is a deferred research question.
Verdict: the core fact is not deferred.
D6 states that foreign thinking blocks are dropped silently and names the preserved-thinking page and the date it was read.
Two supporting parts are weak, though.
First, the claim that "the 400 invalid signature error is the prefix check and compares no account" names no source.
Second, the condition D6 relies on, matching tools, is only partly met by D10's wiring.
`enabledPlugins` sits in each directory's `settings.json`, which D10 links only "as wanted", and MCP servers sit in each directory's `.claude.json`, which D10 never aligns (bound-breaker F6).
The board says that if a copied transcript does not resume, "the session starts fresh".
The record states no such fallback for a copy the API refuses, and its Open risk leaves the first real test to the manual check.

### [MEDIUM] D7's switch is unordered against retryAPIError on the same turn end

**Location:** `_lym/discussion/decision-record.md:183`

**Issue:** `retryAPIError` re-sends a usage-limited turn like any other API error and counts it toward `APIErrorRetries`.
Past those retries the run ends `OutcomeAPIError` (`internal/shuttleengine/apierrorretry.go:46`).
D7 says neither whether a switched turn end is exempt nor whether the watch resets for the relaunched strand.
"Otherwise the limit is handled as today" therefore leaves both paths live.
The lens rated this BLOCKING.
It is kept at MEDIUM because the switch runs in the held-turn branch, before the retry's backoff elapses, so the run is not lost on the first tick; the stated purpose is still at risk on later limited turns.

### [MEDIUM] The orch watcher cannot restart the orch

**Location:** `_lym/discussion/decision-record.md:184`

**Issue:** The watcher's limit handling only renews a creel hold and injects a wake line (`internal/orchengine/usagelimit.go:59-82`).
The resume path, with `endOwnHolder` for the live orch, is `orchcli`'s `resumeOrch` (`internal/orchcli/start.go:796-818`).
D7 does not say how the watcher reaches it.

### [MEDIUM] D2's prune has no input under told geometry

**Location:** `_lym/discussion/decision-record.md:68`

**Issue:** The pick runs in the claude engine, which D4 tells only the worktree's name and kind and the account file path.
Without the hub's worktree list it cannot tell which entries' worktrees are gone, and `PATTERN-cwd-resolution` forbids it to derive that list.

### [MEDIUM] blocked_until and listed-window eligibility disagree

**Location:** `_lym/discussion/decision-record.md:86`

**Issue:** D1 allows an account to list no windows, and D3 step 2 tests only the windows an account lists.
D5 says a future `blocked_until` leaves no headroom in any window, yet an account with no listed windows stays eligible.
D3 step 3's "latest blocking reset" does not cover `blocked_until`.

### [MEDIUM] D5's forged-input bound is understated

**Location:** `_lym/discussion/decision-record.md:142`

**Issue:** Focus directive 2 asks whether each new edge states its bound.
The status-line verb takes any listed name and both windows, so the stated bound, "one listed account's figures in one window", is too narrow.
The other bounds in that directive hold:
- The status line replacing the operator's own is stated in D5's Facts and in the third Auto-mode assumption.
- `set` by any agent is bounded in D7, apart from F16.
- The `CLAUDE_CONFIG_DIR` scrub states what it lets through and names the `env` module as the way forward.

### [MEDIUM] Opened sessions without creel carry no status line

**Location:** `_lym/discussion/decision-record.md:126`

**Issue:** `OpenLaunch` writes `settings.json` and passes `--settings` only when the creel flag directory is non-empty (`internal/shuttleengine/claudeengine/command.go:474-480`).
D5 claims that an opened session's status line still records figures.
Merged from existing-paths F3, walkthrough 4 and bound-breaker F5.

### [LOW] D6's OpenLaunch copy sentence copies nothing

**Location:** `_lym/discussion/decision-record.md:159`

**Issue:** `OpenLaunch` mints a fresh session id, so no transcript exists when it builds the resume line.
Reed's later replay of that line runs under the same stored account.

### [LOW] D3's reset divides 0 by 0 for a retired account

**Location:** `_lym/discussion/decision-record.md:90`

**Issue:** A weight-0 account that has been listed throughout counts as "listed then under an unchanged weight", so it enters the count-over-weight minimum as 0/0.

### [LOW] D7's switch writes a partial counter

**Location:** `_lym/discussion/decision-record.md:175`

**Issue:** The switch does not write the weights and listing, so the next pick's reset rule overwrites the increment of a newly listed account.

### [LOW] Retiring an account moves no worktree off it

**Location:** `_lym/discussion/decision-record.md:43`

**Issue:** D2 drops an entry only when its name has left the list.
A retired account is still listed, so D4 keeps launching worktrees under it, while D1 refuses a pin on it.
Merged from interactions F5 and bound-breaker F9.

### [LOW] Linked config directories defeat the cleaned-path comparison

**Location:** `_lym/discussion/decision-record.md:41`

**Issue:** A `config_dir` that links to `~/.claude` is not recognised as `~/.claude`, so D4 exports it rather than unsetting it.

### [LOW] D7's set bound names the wrong reach

**Location:** `_lym/discussion/decision-record.md:188`

**Issue:** The verb acts on whichever worktree the cwd resolves to, so the bound reaches every worktree any process can change directory into, not only the session's own worktree.

### [LOW] The beams-pin Open risk is the writer's choice, not the parent's

**Location:** `_lym/discussion/decision-record.md:253`

**Issue:** Focus directive 4 asks whether this is a `decision` finding.
Verdict: it is not.
The board's first design line, "every session of one worktree runs under one account", settles which reading holds, and the record already follows it.
No `Decision for claude-account-pick-simple:` note exists on the board, and the record holds no Added-after-Discussion entry.
The line belongs in Auto-mode assumptions, and Q11 closes as an auto-pick.

### [LOW] Rationales re-argue rejected alternatives

**Location:** `_lym/discussion/decision-record.md:71`

**Issue:** Focus directive 5 asks about leanness and relocation.
The compaction rule keeps rejected alternatives out of `## Decisions`.
Four rationale passages argue against an alternative not taken, in D2, D3, D6 and D7.
The completeness check holds for each: the requirement it supports is already stated in its Decision's body, for example D7's "no stopped outcome is recorded".
Cutting them therefore loses no requirement.
The slug-hash alternative appears in no Rejected alternatives bullet.
The Goal's size estimate stays, because the board asks the Discussion to name it.

### [LOW] D10's landing-notes review criterion names no machine or time

**Location:** `_lym/discussion/decision-record.md:299`

**Issue:** The board places both accounts on OSL-1046.
This run's owner machine is neve, and every landing happens on the owner machine.
The record does not say where or when the manual check runs, so it is unclear whether the landing-notes outcome can exist at landing.

## Focus verdicts

- Directive 1, D6: see F4. The core fact has evidence, and two supporting parts and the fallback do not.
- Directive 2, bounds: see F9 and F16. The status-line override and the scrub are bounded.
- Directive 3, the launched-account record across a worktree with no entry, a pinned worktree whose pin is out and a pre-change run: broken for the pinned case (F2) and for the relaunch (F1).
  A worktree with no entry is covered by D4's effective-directory mapping.
  A run from before the change is covered by D8's fallback read.
  At its first launch under the new binary it is engine-built and gets a launched-account record.
- Directive 4, Q11: see F17. The writer can settle it.
- Directive 5, leanness: see F18.
- Directive 6, acceptance criteria: every id D1 to D10 is named as a whole word by at least one bullet.
  Every criterion is checkable behaviour or opens with `Review:`.
  Content gaps are filed under their decisions: F1, F2, F3, F5, F8, F10 and F19.

## Rejected

- Existing-paths F1 rated BLOCKING: kept as F5 at MEDIUM, with the reason given in that finding's block.
- The D5 status line replacing the operator's own as an unbounded override: D5's Facts and the third Auto-mode assumption state it, and it changes only lym panes.
- The Goal's line estimate as a perishable figure: the board asks the Discussion to name its estimate.
