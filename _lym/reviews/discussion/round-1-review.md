---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 (How a session launch reads it) and D7
    summary: A fresh launch with no record picks before any pre-change session resumes, so the later resume keeps the picked record, launches under the new directory and is refused; D7's promise that pre-change sessions resume holds only when the resume comes first.
    fix: In D2, change the fresh-launch bullet so that a fresh launch with no record first searches the known directories (D6) for this worktree's existing transcripts, under the pick lock of D3, and records the directory that holds them instead of picking; only a worktree with no transcript in any known directory picks. Restate D7's pre-change sentence to rest on that rule rather than on launch order, and add a D2 acceptance criterion where a fresh launch in a worktree whose transcripts sit in the default directory records the default directory and advances no count.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2, D3 step 2, D4 first bullet; Scope Out (standalone)
    summary: No decision gates the pick or the record write on a non-empty accounts list, so with the template default [] every fresh launch fails at D3 step 2 and every resume writes a record that D4 then acts on, against Scope's keeps-today's-behaviour promise and the D1 criterion.
    fix: Add to D2 a first rule that with claude.accounts empty, or outside a hub, a launch reads no record, makes no pick, writes no record and exports nothing, so an env module CLAUDE_CONFIG_DIR stands as today; keep an existing record's export only when the list is non-empty. Extend the D1 criterion to a resume launch with an empty list, asserting no record is written and CLAUDE_CONFIG_DIR is neither set nor unset.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 resume bullet against D6 known directories
    summary: D2 searches only the listed accounts' directories and the default for a pre-change transcript while citing D6, whose known set also holds the env module's and the process's CLAUDE_CONFIG_DIR; a hub that sets CLAUDE_CONFIG_DIR through the env module today records the default, D4 unsets the variable and the resume is refused.
    fix: In D2's resume-with-no-record bullet, search every known directory as D6 defines it, the env module's and the process's CLAUDE_CONFIG_DIR included, and record the one that holds the transcript; reword the transcript-found-nowhere sentence to record the directory D6's no-record fallback names (env module, then process, then default). Add a D2 criterion with the transcript under an env module directory.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 resume bullet against D3 step 1
    summary: Only a pick writes the record under the hub lock with the existing-record-wins re-check; a resume's record write has no stated lock, so a concurrent resume and fresh pick in one worktree can overwrite each other and split the worktree over two directories.
    fix: In D2, state that every record write, the resume's included, runs under the same internal/lock write lock D3 names and first re-reads the record, keeping one a concurrent launch wrote whose directory exists. Extend the D3 concurrency criterion to a resume and a fresh launch starting at once in a worktree with no record, ending with one record.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 status line verb
    summary: The new status line verb writes usage/<account>.json from an argument no rule checks against D1's name pattern or the list, so a malformed name escapes the usage directory and an unlisted one writes a stray file, and its stdin figures are taken unbounded, so a forged far-future resets_at pins a window as blocking.
    fix: In D5, state that the verb refuses an argument that does not match [a-z0-9-]+ or names no listed account, writing nothing and printing the line without figures; that it drops a window whose used_percentage is outside 0 to 100 or whose resets_at lies more than the window's own length (five hours, seven days) past now; and that it is reachable by any shell in the hub, bounded to lowering an account's eligibility until that window's reset. Add D5 criteria for a malformed name, an unlisted name and an out-of-range window.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D4 and D5
    summary: Opened sessions (lym shuttle open, reaching Claude.OpenLaunch in internal/shuttleengine/claudeengine/command.go) build their own launch, resume and settings outside Prepare, and no decision says whether they read the record, pick, export CLAUDE_CONFIG_DIR or carry the status line.
    fix: In D4, state that the claude engine's open launch reads and writes the worktree's record by the same D2 rules and exports or unsets CLAUDE_CONFIG_DIR in its launch and resume commands as Prepare does; in D5, state whether the open settings carry the status line command. Add a D4 criterion for an opened session in a worktree recorded on a non-default directory.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D2 last paragraph and D4 third bullet
    summary: A reed relaunch replays the stored command with the directory baked in and reads no record, so after the record is removed or replaced, or its directory is gone, a relaunched strand runs under the old directory and bypasses D2's refusal; D2 also says a reed relaunch reads the record, which D4 contradicts.
    fix: Reconcile D2 and D4 on one rule and state its bound. Either keep D4's replay and say in D2 that a reed relaunch does not read the record, that a record change takes effect only for strands launched after it, and that the way forward for a running strand is to stop and resume it through shuttle; or have the claude engine re-resolve the record at relaunch. Drop reed relaunch from D2's list of launches that read the record if the first.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D6 known directories
    summary: D6's known-directory set omits the directory of a kept record whose account left the list, and lym report and tools/tokencount reach transcripts through the worktree-told reader, which for a landed run whose record died with its worktree falls back to the default; both lose sessions that ran on a non-default account.
    fix: In D6, state that lym report and tools/tokencount locate a run's sessions by searching every known directory rather than through the worktree's record, and add to the known set the directories of worktree records that still exist; state as a bound that history on an account removed from the list and from every record is no longer read. Add a D6 criterion for a landed run whose sessions ran on a non-default account.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3 step 4
    summary: The fallback's candidate whose blocking windows all reset soonest gives different picks for a candidate with several blocking windows, and the fallback has no tie rule, unlike step 3.
    fix: Reword D3 step 4 to take the candidate whose latest resets_at among its blocking windows is earliest, ties going to the earlier entry in the list; extend the D3 fallback criterion with a candidate holding two blocking windows.
  - id: F10
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 record contents and D5 listed-account condition
    summary: The record stores the account name and D5 gates and names the status line by it, so renaming an account silently stops headroom recording for its worktrees, and reusing a name for another directory files one account's figures under another.
    fix: In D5, resolve the status line's account at launch by the record's directory, compared cleaned per D1, against the current list, and pass that entry's current name; with no listed entry for the directory, carry no status line. Add a D5 criterion for a renamed account.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 step 2 and D4 facts
    summary: On macOS every listed account is a candidate with no stated bound on what a never-logged-in account lets through, and the Windows half of the .credentials.json claim has no source in D4's facts.
    fix: In D3 step 2, state the macOS bound, that a listed account never logged in is picked and its sessions stop at Claude Code's login prompt until the operator logs it in; in D4's facts, cite the Windows credential location from the same Authentication docs page, or drop Windows from D3's claim if the page does not name it.
  - id: F12
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks, second bullet
    summary: The Ultra five-hour-window line says D1's per-account window list covers either answer, so it leaves nothing open for the plan writer.
    fix: Move the Ultra five-hour-window line from the decision record's Open risks to the support log's Interview, Turn 1 upstream bullet.
---

### [BLOCKING] Pre-change sessions survive only when a resume precedes every fresh launch

**Location:** `_lym/discussion/decision-record.md` D2 and D7.
**Issue:** D2 lets a fresh launch with no record pick at once, and only a resume with no record searches for an existing transcript.
In the prime after landing, the dobby or any fresh lym session can start before the orch's resume; it picks account B and writes the record.
The orch's resume then keeps that record (its directory exists), launches under B, and `CheckResume` finds no transcript there (`internal/shuttleengine/claudeengine/resume.go`), so it refuses.
A task worktree mid-run at landing breaks the same way when a fresh reviewer spawns before the writer resumes.
Three lenses found it independently.

### [BLOCKING] An empty accounts list still picks, fails and writes records

**Location:** `_lym/discussion/decision-record.md` D2, D3 step 2, D4; Scope.
**Issue:** The template default is `[]`, and nothing gates D2 and D3 on a non-empty list.
As written, every fresh launch in a hub without accounts enters the pick and fails at step 2's no-candidate rule, and every resume with no record writes a record that D4 then turns into an unset of `CLAUDE_CONFIG_DIR`, overriding an `env` module value.
Scope promises standalone runs keep today's behaviour, and D4 and the D1 criterion promise no export with no accounts; the decisions do not carry that promise.

### [MEDIUM] D2's resume search set is narrower than the D6 set it cites

**Location:** `_lym/discussion/decision-record.md` D2 resume bullet, D6.
**Issue:** The board's "Built since" section says a hub can set `CLAUDE_CONFIG_DIR` through the `env` module today, so pre-change transcripts may live there.
D2 searches only the listed accounts and the default, records the default, and D4 unsets the variable, so those resumes are refused.

### [MEDIUM] A resume's record write has no lock or re-check

**Location:** `_lym/discussion/decision-record.md` D2 against D3 step 1.
**Issue:** A resume that read no record and a concurrent fresh pick can each write the record; the later write wins and the worktree's sessions split over two directories, which D2 rules out.

### [MEDIUM] The status line verb's argument and input are unbounded

**Location:** `_lym/discussion/decision-record.md` D5.
**Issue:** The verb is a new edge any shell in the hub can reach.
Its account argument names a file path with no check against D1's pattern or the list, and its stdin figures merge with later-reset-wins and no range check, so one forged call can hold an account out of every pick until a far-future reset.
D5 states neither bound, which the rubric's attack-surface rule requires.

### [MEDIUM] Opened sessions are a launch path no decision covers

**Location:** `_lym/discussion/decision-record.md` D4, D5.
**Issue:** `Claude.OpenLaunch` in `internal/shuttleengine/claudeengine/command.go` loads the config and builds its own launch and resume commands and settings, apart from `Prepare`.
D4 and D5 name only the session the engine prepares, so an opened session in a worktree recorded on account B would run under the default directory, against D2's every-session rule.

### [MEDIUM] Reed relaunch replays a stale directory and bypasses D2's refusal

**Location:** `_lym/discussion/decision-record.md` D2 last paragraph, D4 third bullet.
**Issue:** D2 lists a reed relaunch among launches that read the record, while D4 has it replay the stored command and read nothing.
After the operator removes the record, the one way the Auto-mode assumptions give to change an account, or after a fresh launch re-picks over a gone directory, relaunched strands keep the old directory, and the refusal D2 promises for a gone directory never runs.

### [MEDIUM] D6's known directories lose delisted accounts and landed runs

**Location:** `_lym/discussion/decision-record.md` D6.
**Issue:** D2 keeps a record whose account left the list, but D6's known set holds only listed accounts, the `env` value, the process value and the default.
`tools/tokencount` locates sessions through the worktree-told `Transcripts.LocateSessions` (`tools/tokencount/reads.go`), and D2's record dies with the worktree, so a landed run's sessions on a non-default account fall back to the default and are missed.

### [LOW] D3 step 4's ranking and ties are undefined

**Location:** `_lym/discussion/decision-record.md` D3 step 4.
**Issue:** With A blocked until +1h and +3d and B blocked until +2d, an earliest-reset reading picks A, which is still blocked after the hour; a when-all-clear reading picks B.

### [LOW] The record's account name drifts from the list

**Location:** `_lym/discussion/decision-record.md` D2, D5.
**Issue:** D5 passes the record's stored name, so a renamed account loses headroom recording without notice and a reused name files figures under the wrong account.
D1 already cleans every compared directory, so resolving by directory at launch costs nothing new.

### [LOW] macOS candidates are unbounded and the Windows claim is unsourced

**Location:** `_lym/discussion/decision-record.md` D3 step 2, D4 facts.
**Issue:** D3 states the bound of the Linux stat but not what the macOS skip lets through, and D4's evidence names Linux and macOS only.

### [NIT] An open risk the design already settles

**Location:** `_lym/discussion/decision-record.md` Open risks.
**Issue:** The line itself says the per-account window list covers either answer; it is deliberation narrative with no requirement for the plan writer, so it belongs in the support log.

## Focus notes

Each focus entry was reached.
D5's statusLine replacement of the operator's own command states its bound in D5 itself, not only in Auto-mode assumptions: not a defect.
D3's macOS Keychain claim rests on D4's sourced facts, and D1's Ultra open risk defers no fact the design rests on: neither is a deferred research question.
D4's unset-in-default criterion, the D7 criterion and the D8 criterion are each testable through the launch and resume commands: not defects.
D9's operator wiring steps carry the board's requirement that the task state how the second account is wired, and the Rationale paragraphs are Decision rationale the spec allows: neither is relocated.
