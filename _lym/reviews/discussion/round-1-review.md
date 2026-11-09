---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:172
    summary: lym adopt --here ends owned_here without reading the record, so lym s after an adopt that stopped or died after its owner write skips the deploy, beam and check steps and runs beam start on the old lym
    fix: In D10, replace "When no other machine owns the subject, it ends ok with the outcome `owned_here`, spawning no worker and reading or writing no record." with a rule that reads the record first; when an unfinished record names the subject, `--here` is `lym adopt <subject>` under D1 and D2 (follow or resume), whoever the owner reads as; only with no such record does it end `owned_here`, spawning no worker and writing no record. Narrow D10's rationale to "An owned-here `lym s` never meets another subject's unfinished record." as it stands, and add a D10 acceptance criterion that `lym s` after a worker killed at `deploy` resumes at `deploy` and runs no `beam start` until the adopt ends.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:92
    summary: D4 covers only the subject's own worktree, so during lym adopt root a beam with no owner of its own reads the prime's foreign owner and its orch watcher stops the orch and its dobby runs D11's archive-and-exit on the adopting machine
    fix: In D4, define the covered worktrees as the subject's own worktree and every beam worktree whose owner it reads through `hubgeom.ParentOf` from the subject (a beam with no owner recorded under an adopted `root`); state that the watcher and dobby of each such beam read the record the same way and stay up and idle until the subject's owner write stands. Add an acceptance criterion that during `lym adopt root` the watcher and dobby of a beam with no owner of its own neither stop, exit, archive nor start a pass.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:61
    summary: lym adopt --discard checks the worker dead and removes the record without adopt.lock, so a concurrent adopt's fresh running record can be deleted and the live worker runs with no record and no D4 cover
    fix: In D2, state that `lym adopt --discard` takes `adopt.lock` non-blocking, refuses naming `lym beam status` when the lock is held, and removes the record only while it holds the lock; extend the D2 discard acceptance criterion with a discard that races a starting worker and leaves its record standing.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:55
    summary: A resume skips reconcile however old the record is, then the owner step's takeover write can take the subject from a third machine that adopted it since, on stale pairs, stops and pause set
    fix: In D2, make a resume whose first step not done is `owner` or earlier start at `reconcile` (the steps are safe to re-run, per Constraints), so its owner write always follows a fresh reconcile; keep the skip only for resumes past `owner`. Replace the bound "A resume does not re-read origin for the steps before its first step not done" with that rule, and add an acceptance criterion that a resume before `owner` re-runs the reconcile.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:39
    summary: The D1 bound says the worker takes its request only from argv the caller and resume build, but the hidden lym adopt worker verb is callable by any shell or agent and then skips the caller's validation and D2's resume-or-refuse rule
    fix: In D1, state that the worker runs the caller's validation itself and applies D2's rules against the standing record (refusing other subjects, resuming same subjects) before taking any step, so a direct call can skip nothing the caller checks; rewrite the bound to say a direct call is one more caller, bounded by the same validation, D2's subject match and `adopt.lock`.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:183
    summary: The D10 bound claims beam start then refuses naming lym adopt, but with origin unreachable beam start's prepare only warns, so a stale Bolt that reads this machine as owner brings the orch and dobby up on a subject another machine owns
    fix: In D10's bound, replace "and `beam start` then refuses naming `lym adopt`" with the true outcome, that with origin unreachable both `--here` and `beam start` judge the owner on the local Bolt, so a stale Bolt starts the beam here as `lym beam start` does today, and the next reachable Bolt pull hands the subject to D11's old-owner path; or make `--here` end non-zero naming `lym adopt <subject>` when origin is unreachable, and say which in D10.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:194
    summary: D11 pushes archive tags for live runs and for root and beam slugs, but the report engines and records reads treat every archive/<slug>/ tag as an archived run, and the record is silent on how they read the new tags
    fix: In D11, state how the archive-tag readers in `internal/reportengine` and `internal/fabricengine` treat D11's tags, either by naming them apart (a distinct prefix such as `archive-owner/<slug>/<sha12>`, which D12 then reads) or by a rule those readers apply to skip them; add an acceptance criterion that the run report lists no run for a D11 tag of a live run or of `root`.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:204
    summary: D11's rationale claims a tag push moves no branch PATTERN-owner-machine reserves, but that entry's rule text bars a push by a machine that reads the entry owned elsewhere, with no tag exemption cited
    fix: In D11's rationale, cite the `PATTERN-owner-machine` rule text and state why an archive-tag push is outside it, with the teardown's archive push of an owned-elsewhere pair as precedent; if it is not outside it, add a `PATTERN-owner-machine` change bullet under `## PATTERN changes` that exempts the old owner's archive-tag push.
  - id: F9
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:25
    summary: The worker registers under runproc.HubOwner, and lym fabric anchor's EndDobbies ends every HubOwner role but push and sync, killing an adopt worker mid-step
    fix: In D1, state that the adopt role is spared wherever hub-owner processes are ended with `runproc.EndSparing` (the anchor's dobby end), or add to D1's bound that such an end interrupts the worker and D2's resume picks it up.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:27
    summary: No decision says which exit code a stopped record gives, yet D10's claim that a stopped adopt starts nothing rests on a non-zero exit, and today a held-back project exits 0
    fix: In D1, add that the caller exits non-zero when the record ends `stopped` or `interrupted`, and 0 only when it ends `done`.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:44
    summary: D1's caller prints the envelope the worker wrote, but D2's list of the record's contents holds no envelope, and a resume's envelope coverage of earlier steps is unstated
    fix: In D2's list of what the record holds, add the end envelope the worker writes at `done` or `stopped`, built from the outcomes of every step the record lists as done, earlier processes' steps included.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:28
    summary: A caller whose worker lost the lock can read the previous done record of the same subjects before the winner writes, and print a stale success envelope
    fix: In D1, state that the caller follows another pid's record only when it is unfinished and its start is later than the caller's own start; until such a record appears, it keeps waiting under the same wake rules.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:49
    summary: The winner writes the record running at reconcile before any step runs, which read literally wipes the done steps and stored inputs a resuming worker needs
    fix: In D2, change the sentence to say a fresh worker writes the record `running` at `reconcile`, and a resuming worker rewrites only the pid, start ticks and state, `running` at its first step not done, keeping the done steps and stored inputs.
  - id: F14
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:56
    summary: The resume's flag merge can join a recorded --keep-local with a call's --drop-local, a pair adopt refuses, leaving --discard as the only way on
    fix: In D2, state that a call's `--keep-local` or `--drop-local` replaces the record's other one rather than adding to it.
  - id: F15
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:57
    summary: A resume that restarts at reconcile re-runs the pause step and adopt-pauses again a run the operator already resumed by hand after the first pause step
    fix: In D5, state that the `pause` step skips every slug the record already lists as paused, so a re-run pauses only slugs it has not paused before.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:92
    summary: D4 tests whether the record names the subject, but a --all record stores the flag, not a subject set
    fix: In D2, add the resolved subject set to the record's contents, written at `reconcile`, and have D4 test the subject against that set.
  - id: F17
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:133
    summary: The D5 bound says an older check across the upgrade can lift a legacy hold once, but the old green end also brings up dobby and orch, starts first passes and runs lym loom resume on owner-halted children
    fix: In D5's bound, state the full effect of an old green end across the upgrade, that it can lift the legacy holds it lists, bring up the subject's dobby and orch, start the lifted runs' first passes and resume owner-halted children, once.
  - id: F18
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:138
    summary: D6 claims no run goes on under the old lym, but D5 pauses no front task, so the subject's dobby can start an open front task after the owner write during a failed deploy
    fix: In D6, narrow the claim to the runs D5 pauses, and state that an open front task under the subject can start once the owner write stands, before the deploy.
  - id: F19
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:192
    summary: D11 says the owner pause comes first so no pass starts while it archives, but a failed owner pause gives the run a full pass on the next probe tick while the archive retries
    fix: In D11, state that a dobby whose owner pause failed retries the pause before the archive at each probe and starts no pass of the run while the subject reads owned elsewhere, or add to D11's bound that a failed owner pause lets the run's full pass go on as today.
  - id: F20
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:207
    summary: D11's bound puts a commit after the archive in the pause's grace, but the owner pause waits out its grace and ends the run's processes before returning, so the archive runs after the grace
    fix: In D11's bound, replace "A commit made after the archive, in the pause's grace, stays local." with "A commit made by hand in a worktree after the archive stays local."
  - id: F21
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:212
    summary: lym adopt --archives also lists every teardown and --drop-local archive tag under the subject, since a torn-down slug's origin branch is gone, and calls them the old owner's archives
    fix: In D12, restrict the listing to tags D11 writes, by the distinct name F7 settles or by listing only slugs whose origin branch exists, and say which.
  - id: F22
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:212
    summary: D11 archives code and weft tips under one tag shape in two repositories, but D12 compares each tag with the slug's origin branch without naming the repository or side
    fix: In D12, state that each repository's archive tags are read from that repository's origin and compared with that repository's branch of the slug (the code branch for warp tags, the weft branch for weft tags).
  - id: F23
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:261
    summary: The Review criterion lists what the PATTERN-dobby-runs background must describe but omits its Dobby and orch pair rule, which D7's dobby-only beam step now departs from
    fix: In the Review acceptance criterion, add that the `PATTERN-dobby-runs` background's Dobby and orch pair rule names the adopt beam step's dobby-only start under `--no-orch`.
  - id: F24
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:230
    summary: The third Auto-mode assumption restates D11's rationale line that the pause comes first so no pass starts while it archives
    fix: Delete the D11 rationale line "The pause comes first so the dobby starts no pass while it archives." and keep the Auto-mode assumption, which also records the departure from the board's order.
---

### [BLOCKING] `lym s` cannot resume an adopt that stopped after its owner write

**Location:** `_lym/discussion/decision-record.md:172`

**Issue:** D3 puts `owner` before `deploy`, `beam` and `check`, so the case the board names, an adopt dying mid-deploy, now happens after the owner write.
On the entry point D10 makes normal, `--here` then reads the subject owned here and ends `owned_here` without reading the record.
The shortcut goes on to `beam start` on the undeployed lym, and no adopt check ever starts.
The Goal's "resumes after a break" fails on the path that matters most.
Interactions and walkthrough both raised it.

### [BLOCKING] D4 leaves beams that inherit the prime's owner uncovered during `lym adopt root`

**Location:** `_lym/discussion/decision-record.md:92`

**Issue:** `PATTERN-owner-machine` has a beam with no owner recorded read its parent's owner.
During `lym adopt root`, the reconcile makes such a beam read foreign-owned, but D4 covers only the subject's own worktree.
The beam's orch watcher (`internal/orchengine/owner.go` `ownerTick`) stops its orch, which is the board's first finding again.
Its dobby (`internal/dobbyengine/loop.go:494`) is not under adopt either, so under D11 it owner-pauses, pushes archive tags and exits, on the machine that is adopting.
D7 brings up only the subjects' dobbies, so those beams stay down after the adopt.

### [MEDIUM] `--discard` races a starting worker

**Location:** `_lym/discussion/decision-record.md:61`

**Issue:** The discard judges liveness and deletes without `adopt.lock`.
A worker that takes the lock between the two steps loses its fresh record.
D4 then reads no record mid-adopt, and the orch watcher stops the orch.
Interactions and bound-breaker both raised it.

### [MEDIUM] A stale resume can take the owner from a third machine

**Location:** `_lym/discussion/decision-record.md:55`

**Issue:** A resume reuses the record's stops, moved runs and pause set without reconciling.
`SetOwners` with `Takeover: true` writes over any owner, so a resume days later takes a subject a third machine has adopted since.

### [MEDIUM] The hidden worker verb's bound does not hold

**Location:** `_lym/discussion/decision-record.md:39`

**Issue:** A cobra verb is reachable from any shell, so the claim that only the caller and the resume build its argv bounds nothing.
A direct call skips today's validation in `adopt` (`internal/adoptcli/adopt.go:208-233`) and D2's subject match.

### [MEDIUM] The D10 stale-Bolt bound misstates what `beam start` does

**Location:** `_lym/discussion/decision-record.md:183`

**Issue:** `orchCLI.prepare` (`internal/orchcli/start.go:446-447`) only warns when origin is unreachable.
So the stale Bolt that skipped the adopt also lets `beam start` bring the beam up, not refuse.

### [MEDIUM] Existing archive-tag readers would read D11's tags as runs

**Location:** `_lym/discussion/decision-record.md:194`

**Issue:** `selectRuns` in `internal/reportengine/runs.go:139` indexes every tag `ArchiveTagSlug` parses as a run's archive, and so do the cost, timing and raddle reports.
D11's tags of live runs and of `root` and beam slugs land in that index, and the record says nothing about it.
Existing-paths and bound-breaker both raised it.

### [MEDIUM] The PATTERN-owner-machine claim in D11's rationale cites no rule text

**Location:** `_lym/discussion/decision-record.md:204`

**Issue:** The entry's rule bars a "push" on a machine that reads the entry owned elsewhere.
The rationale asserts a tag push is outside it, but cites neither the text nor a precedent.

### [LOW] The anchor's dobby end kills the hub-owned worker

**Location:** `_lym/discussion/decision-record.md:25`

**Issue:** `internal/fabriccli/anchor.go:468` ends the prime's hub-owner processes, sparing only push and sync, through `runproc.EndSparing` (`internal/fabriccli/anchordeps.go:258`).

### [LOW] The exit code of a stopped record is unstated

**Location:** `_lym/discussion/decision-record.md:27`

**Issue:** D10's "a stopped adopt starts nothing" needs a non-zero exit, and `adopt` today returns no error for a held-back project.

### [LOW] The record holds no envelope for the caller to print

**Location:** `_lym/discussion/decision-record.md:44`

**Issue:** D1 prints the envelope the worker wrote, but D2's contents list has none.

### [LOW] A losing caller can follow a stale done record

**Location:** `_lym/discussion/decision-record.md:28`

**Issue:** The pid check shows only that the record is not the caller's own, not that it is current.

### [LOW] The record write before any step reads as wiping a resume's state

**Location:** `_lym/discussion/decision-record.md:49`

**Issue:** The sentence covers every winner, a resuming one included.

### [LOW] The flag merge can yield both local-commit flags

**Location:** `_lym/discussion/decision-record.md:56`

**Issue:** `adopt` refuses `--keep-local` with `--drop-local` (`internal/adoptcli/adopt.go:213-215`).

### [LOW] A restart at reconcile re-pauses hand-resumed runs

**Location:** `_lym/discussion/decision-record.md:57`

**Issue:** The Constraint "pauses only unpaused slugs" counts a run the operator resumed as unpaused, so the adopt stops it again.

### [LOW] D4's subject test is undefined for an `--all` record

**Location:** `_lym/discussion/decision-record.md:92`

**Issue:** D2 stores the request and matches `--all` by the flag, so the accessor has no subject set to test against.

### [LOW] The D5 legacy-check bound understates the old green end

**Location:** `_lym/discussion/decision-record.md:133`

**Issue:** `greenEnd` (`internal/adoptcli/green.go`) also calls `BeamUp`, starts lifted runs and resumes halted children.

### [LOW] D6's no-run claim misses front tasks

**Location:** `_lym/discussion/decision-record.md:138`

**Issue:** D5 skips front tasks, and D4's cover ends at the owner write.

### [LOW] A failed owner pause lets a pass start while D11 archives

**Location:** `_lym/discussion/decision-record.md:192`

**Issue:** `ownedElsewhere` records `OwnerPauseFailed`, and `planPass` then gives the run a full pass on each probe tick while no pause covers it (`internal/dobbyengine/owned.go:39-43`, `internal/dobbyengine/passes.go:124-127`).

### [LOW] D11's bound misplaces the window

**Location:** `_lym/discussion/decision-record.md:207`

**Issue:** `tenderengine.Pause` waits out the grace before it returns, so no archive runs inside the grace.

### [LOW] `--archives` lists teardown archives as the old owner's

**Location:** `_lym/discussion/decision-record.md:212`

**Issue:** Teardown and `--drop-local` write the same tag shape, and their slugs' origin branches are gone.
Existing-paths and bound-breaker both raised it.

### [LOW] D12 names neither repository nor branch side

**Location:** `_lym/discussion/decision-record.md:212`

**Issue:** `ArchiveTagName` carries no side, and weft tags live in the weft repository's origin.

### [LOW] The Review criterion omits the Dobby and orch pair rule

**Location:** `_lym/discussion/decision-record.md:261`

**Issue:** The `PATTERN-dobby-runs` background's Dobby and orch pair rule says a dobby starts only with its orch.
The declared overview text adds the `--no-orch` exception, but the Review criterion does not require the background rule to carry it.

### [NIT] The pause-before-archive reason appears twice

**Location:** `_lym/discussion/decision-record.md:230`

**Issue:** The Auto-mode assumption and D11's last rationale line say the same thing.

## Handler notes on the focus

- D11's "named interval" is a choice, not a fact, so it is no deferred research question.
  The probe it rides already backs off per `PATTERN-daemon-wakeups`, and Plan names the constant.
- No Decision files or promotes a task, or edits this task's own board entry.
  The red end's fix task is today's behaviour, which the Out list keeps.
- `Open risks: None` holds.
  No `Decision for adopt-robust:` note stands, no other board entry or run touches adopt, and no commit since the task was filed touches the adopt, dobby, orch, pause-record or shortcut packages.
- D4's read without a lock is safe: `internal/state` replaces the file atomically.
- Leanness: the Decisions carry no deliberation narrative that belongs in the support log, beyond F24.

## Rejected

- Walkthrough's LOW, that `--here`'s Bolt pull before any record exists lets the watcher and dobby of the machine about to adopt act on a foreign owner: the pull brings in a foreign owner only where this machine's Bolt read the subject as its own. That machine is the old owner, for which D11's archive-and-exit is the intended behaviour, and the `lym s` chain's `beam start` brings the orch back after the adopt.
