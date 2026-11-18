---
verdict: BLOCKING
findings:
  - id: F20
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:202
    summary: D7's relaunching mark has no failure, restart, sweep or concurrent-stop path, so a Wait process that dies between steps 1 and 4 leaves a record attach reads as live forever, a sibling Start's orphan sweep deletes the run directory, and a stop during the window is overwritten by step 4
    fix: "In D7's numbered relaunch list and the `## PATTERN changes` bullet, bound the relaunching mark. (1) In step 1, say a failed strand removal clears the mark back to running over the live strand, and the limit is then handled as today. (2) Add to step 1 that the orphan sweep keeps a run directory whose record reads relaunching. (3) Add after step 4: the mark records when it was written; an attach or Wait that reads a relaunching record with no tracked strand, once the startup timeout has passed since the mark, either finishes the relaunch from step 2 itself or ends the run as a strand that died, done when every output file exists. (4) Add to step 4: it re-reads the run record under the record lock before writing the new GUID, and when a stop verb call has made it terminal meanwhile it removes the new strand instead of writing running. Reword the PATTERN changes bullet to say a relaunching record reads live until the new strand's GUID is written or the relaunch bound passes, and that the orphan sweep keeps it; keep the bullet within the PATTERN rule-text cap. Add acceptance criteria under D7: an attach after the Wait process is killed between steps 1 and 4 finishes or ends the run once the bound passes; a sibling Start's orphan sweep during a relaunch keeps the run directory; a StopStrand during a relaunch leaves the run stopped and no strand up."
  - id: F21
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:207
    summary: D7 step 4 writes a new strand GUID into the run record, so every caller holding the old GUID (the tackler marker's StopStrand and attach, webster's recover-batch and run-level reclaims, the sampler and dobby stops, FindRun) finds no run, removes nothing and can launch a second agent beside the relaunched one
    fix: "In D7, choose and state one of two mechanisms. Either the relaunch keeps the strand's GUID: step 1 ends the session's process without removing the strand, and step 4 has reed replace the strand's stored Cmd and ResumeCmd and relaunch its pane under the same GUID, with F20's mark and bound applying to that window. Or the run record keeps every superseded GUID, and FindRun, StopStrand and attach match any of them, the stop verb removing the current strand. Add an acceptance criterion under D7: StopStrand and attach given the pre-relaunch GUID reach the relaunched run; the stop records stopped and removes the new strand."
  - id: F22
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:204
    summary: D7's relaunch builds its resume launch through Prepare with the run's spec, so the resumed session's first turn is the skill-load message or the whole task prompt, the deferred prompt line is never sent, and the new strand's startup gates never run because the record already holds Started
    fix: "In D7 step 2, state the relaunch spec: it carries ResumeSessionID, no skills, since the resumed transcript holds their loads, and in place of the run's task prompt one continue line telling the session its account was switched and to carry on where it stopped. In step 4, state that the new strand passes the start path's startup gates (awaitStartup) and that any Launch.PromptLine is sent and RunState.PromptLine and the prompt offset are persisted, as launch does for a new run, before the GUID is written. Extend D7's first acceptance criterion: after the relaunch the session's first new turn is the continue line, neither a skill load nor the task prompt, and the new strand passed its startup gates."
  - id: F23
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:170
    summary: D6 has the resume check record its copy beside the run directory's launched-account record, but on the start path, which the orch's resume also takes, the check runs before the run directory exists and is given none, so the refused-resume fallback the board asks for can never fire there
    fix: "In D6, replace 'The check records a copy it made beside the run directory's launched-account record (D4).' with: the resume check reports whether it copied; on the start path the runner carries that fact into the launch and writes it into the new run directory beside the launched-account record (D4) after the directory is created, and in D7's relaunch the relaunch writes it. Add an acceptance criterion under D6: a shuttle resume and an orch resume through StartSession whose check copied the transcript record the copy in the new run's directory, and a first-turn API error other than a usage limit then starts the session fresh."
  - id: F24
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    carries: F6
    location: _lym/discussion/decision-record.md:219
    summary: The watcher's orch restart through resumeOrch only ends the holder process and starts a session, leaving the old orch strand in reed, the new process unregistered and the orch state and watcher bound to the old strand GUID
    fix: "In D7's orch paragraph, replace 'a restart through resumeOrch, which ends the orch's own live holder when its resume is refused for one, then resumes once more' with: a restart through the orch start path's relaunch branch (the StartRelaunch case in internal/orchcli/start.go): withdraw the orch's pending mail, remove the old orch strand, launch through launchOrch with the session resumed, register the new process, and save the state reset onto the new strand GUID, without spawning a second watcher. State that the fresh start capability takes the same branch with a fresh launch. Extend D7's orch acceptance criterion: after the restart reed holds no old orch strand, the orch state names the new strand and the new process is registered as the worktree's orch."
  - id: F25
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F9
    location: _lym/discussion/decision-record.md:151
    summary: D5's forged-input bound says the effect is limited to the pick falling back to D3's step 3, but D7's switch judges eligibility by D3's step 2 too, so forged figures also make limited worktrees wait limits out or switch onto an exhausted account
    fix: "Replace D5's sentence 'The effect is bounded to the pick falling back to D3's step 3 and to no blocked_until being written.' with: forged figures also steer D7's switch: forged high figures on every other account make a limited worktree switch nothing and wait its limit out, up to each forged window's length; forged low figures on an exhausted account make a switch take it, at most once per account before the limit hit's blocked_until rules it out. No forged input writes or clears a blocked_until."
prior:
  - id: F1
    status: resolved
    answer: D7 now defines the relaunch as a mechanism in internal/shuttleengine through the stop verb's relaunch mode, Prepare, then the resume check, with a PATTERN changes bullet and the extended acceptance criterion. The new defects in that mechanism are raised as F20 to F22.
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: open
    answer: The watcher is now told a restart capability backed by resumeOrch, as the fix asked, but resumeOrch alone leaves the old strand and the orch state unbound; carried as F24.
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: open
    answer: The bound now covers both windows of every account, but still limits the effect to the pick's step 3 while D7's switch also reads the forged figures; carried as F25.
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
    answer: Links are resolved before comparing; the fixer's reason for comparing a directory that does not resolve, rather than refusing it, holds, since a refusal would stop every launch in the hub over a retired account's missing directory.
  - id: F16
    status: resolved
  - id: F17
    status: resolved
  - id: F18
    status: resolved
  - id: F19
    status: resolved
---

Severity follows the rubric's mapping onto the four values.
BLOCKING is a mechanism that cannot work as written, or a stated guard that leaves a run stuck or duplicated.
MEDIUM is a defect that leaves a board requirement partly unmet or a stated bound wrong, with a working path around it.
This is an incremental round, so a new finding is raised only at BLOCKING and only on a changed hunk.
Every new finding sits in D6 or D7 text the round-1 fixes wrote.

### [BLOCKING] D7's relaunching mark has no bound and no failure, restart, sweep or stop path

**Location:** `_lym/discussion/decision-record.md:202`, with the PATTERN changes bullet at `:290`.

**Issue:** Step 1 marks the run record relaunching and removes the strand.
The declared PATTERN change then has attach and `Wait` read that record as live while no strand is up.
The mark has no bound, so several paths break:

- **Step 1 fails.** `Stop`'s own contract (`internal/shuttleengine/stop.go:39-40`) leaves the record written over a live strand when the removal fails. "A failure in steps 2 to 4" does not cover step 1, so the mark sits over the old process.
- **The Wait process dies between steps 1 and 4.** A crash, a hibernate or a binary handover is enough. The record stays relaunching with no strand, nothing finishes the relaunch, and a restarted driver's attach reads a live run until the run's deadline.
- **A sibling `Start` sweeps the run.** Every start runs the opportunistic orphan sweep (`run.go:604`). The sweep removes a directory older than the age guard whose `StrandGUID` reed does not track (`internal/shuttleengine/rundir.go:355-363`). It never reads the outcome, so it deletes the relaunching run's `run.json`, events file and launched-account record.
- **A stop during the window.** `stopRecorded` writes stopped from another process (`stop.go:87-101`). Step 4 then saves its in-memory state with the new GUID and running, overwriting the stop.

This is also the rubric's attack-surface question: the PATTERN change downgrades the liveness guard and states no bound.
The finding merges the bound-breaker lens's F-B, the existing-paths lens's F-A and F-C, and the handler's own reading.

### [BLOCKING] D7 step 4's new strand GUID orphans every caller holding the old one

**Location:** `_lym/discussion/decision-record.md:207`

**Issue:** Step 4 writes the new strand's GUID into the run record.
`FindRun` matches only `rs.StrandGUID == guid` (`internal/shuttleengine/rundir.go:288`).
`StopStrand` of a GUID with no record removes only that GUID's strand, which step 1 already removed (`stop.go:76-79`).
The sheddriver tackler's reclaim stops or attaches by its marker's stored GUID (`internal/sheddriver/tackler.go:166-171`).
The existing-paths lens found the same pattern in webster's recover-batch and run-level reclaims and in the sampler and dobby stops.
After a relaunch, a deadline stop leaves the relaunched agent running unowned, and an attach by the old GUID finds no run.
A reclaim that then starts the step again puts a second agent beside the first, which is the duplicate-agent hazard `run.go:690-704` warns of.

### [BLOCKING] D7's relaunch re-types the task or a skill load and skips startup and the deferred prompt line

**Location:** `_lym/discussion/decision-record.md:204`

**Issue:** Step 2 builds "a fresh resume launch" through `Prepare`.
`Prepare` always puts a first turn on the launch line, even with `--resume` (`internal/shuttleengine/claudeengine/command.go:383-387`).
That first turn is the `prompt.md` pointer, or the skill-load message when the spec names skills, which then moves the pointer to `Launch.PromptLine` (`claudeengine.go:281-285`).
Only the start path's `launch` sends `PromptLine` after startup and persists it.
Step 4 says only that the strand starts and its GUID is written.
`Wait` seeds `started` from the persisted `Started` fact (`wait.go:277-286`), so no startup probe runs for the new pane.
The result depends on the run's spec:

- **A run with skills**, which most roles have, loads them again and ends its turn held with no task.
- **A run without skills** gets its whole task prompt again in the middle of its session.

Neither continues the limited turn, which is D7's stated purpose.
The existing-paths lens's F-D, the missing startup gates, is merged here.

### [BLOCKING] D6's copy record has no run directory on the start path, so the board's fresh-start fallback never fires there

**Location:** `_lym/discussion/decision-record.md:170`

**Issue:** D6 makes the fallback depend on "a resume whose check copied the transcript", recorded "beside the run directory's launched-account record".
On the start path, `checkResume` runs at `internal/shuttleengine/run.go:589`, before `launch` creates the run directory (`:645`) and calls `Prepare` (`:650`).
`CheckResume(sessionID, workdir)` takes no run directory (`engine.go:357`).
The orch's resume reaches the same path through `StartSession` (`internal/orchcli/start.go:806-808`).
That path covers D9's prime orch whose account changed, where the board asks for "otherwise the session starts fresh".
There the copy can never be recorded, so a copied resume the API refuses fails turn after turn instead of starting fresh.
The bound-breaker lens raised this as BLOCKING, and the interactions and walkthrough lenses as MEDIUM.
It stays BLOCKING because the board-required fallback has no carrier on its main entry points.

### [MEDIUM] The watcher's orch restart through resumeOrch leaves the old strand and state behind (carries F6)

**Location:** `_lym/discussion/decision-record.md:219`

**Issue:** `resumeOrch` ends the holder through `endOwnHolder` and calls `StartSession` (`internal/orchcli/start.go:801-818`).
The rest of a restart happens around `launchOrch` in the start path's relaunch branch (`start.go:692-744`): withdrawing mail, `RemoveStrand` of the old orch strand, `registerOrchProcess`, `ResetForFreshLaunch` and `saveFreshLaunch` with the new GUID.
With `resumeOrch` alone, the old pane stays in reed, the new process is unregistered, and the watcher's state names the old strand.

### [MEDIUM] D5's forged-input bound omits D7's switch (carries F9)

**Location:** `_lym/discussion/decision-record.md:151`

**Issue:** D7's switch takes "by D3's steps 1 and 2, an eligible candidate" (`:191`), so forged figures steer it as well as the pick.
Forged high figures on every other account make limited worktrees wait out their limits.
Forged low figures on an exhausted account send a switch onto it.
The bound limits the effect to D3's step 3 only.

## Rejected

Each item below is valid as far as the lenses report it, but sits below this incremental round's bar: a new finding must be BLOCKING, and each of these leaves a working path.
None is a false positive; the fixer may still address them while rewriting the D6 and D7 text above.

- The `blocked_until` write is not ordered before D7's switch reads D4's resolution, so a beams-pinned switch can write an entry without relaunching. Raised by interactions 2, walkthrough 4 and bound-breaker F-F; at worst the limit is handled as today, with one extra count.
- The resume check and `Prepare` resolve the account separately, so a `set` between them can copy into one account and launch under another (interactions 3). The window is narrow, and the resume falls back to today's no-transcript handling.
- D5's `OpenLaunch` clause "also with no creel flag directory" and its acceptance clause describe a case that cannot occur, since an engine with no board directory loads no machine layer (interactions 4, bound-breaker). The text is harmless.
- Removing or changing a pin leaves worktrees made under it unweighed until an adopt or a `set` (interactions 5). They run as today.
- D6's fallback fires on any first-turn API error, transient ones included (bound-breaker F-C). This is a cost, not a broken mechanism.
- D3's "replaces any entry" against the one-increment acceptance criterion for two concurrent picks (bound-breaker F-D). This is a counter skew.
- Whether the told worktree-names reader lists the prime (bound-breaker F-G). The writer can state it in the D2 text.
- The copy runs before the live-holder test (bound-breaker F-H). It overwrites only an older copy, with the newest copy kept as its source.
- A reed relaunch of a no-account launch after an `env` change runs under another directory than the recorded one (bound-breaker F-I). This is the documented relaunch behaviour, under an operator's own change.
