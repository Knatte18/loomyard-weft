---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D5 (Bolt set-aside and replay), D3, Acceptance criteria Neve's case
    summary: D5 assumes per-entry Bolt files, but every Bolt commit rewrites the shared board.json and the minute-stamped README.md, so the set-aside test matches no real commit and a cherry-pick replay of any kept Bolt commit near-certainly conflicts, making almost every Bolt divergence a D3 stop of every scoped entry
    fix: Rewrite D5's Bolt half. Define the Bolt set-aside by the board.json entries a commit changes (a commit whose every changed board.json entry, and every design-<slug>.md it touches, belongs to an ended entry). Replace the cherry-pick replay on Bolt with a semantic replay - re-apply each kept commit's entry-level changes through boardengine onto origin's board.json and re-render README.md and the design files, so README.md and untouched entries never conflict; a conflict is only the same entry field changed on both sides. Add an acceptance criterion - two machines each with one status commit of different entries replay without a stop.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D3 step 2, D6 first paragraph
    summary: An entry counts as ended when it is gone from origin's Bolt tip, so an entry origin never held, such as one created while Bolt pushes failed with a 403 as in issue 737, is ended and its live but parked pair is torn down and its resume refused
    fix: In D3 step 2, define gone as removed from origin - the entry is absent from origin's Bolt tip but present in an origin Bolt commit this machine's local Bolt already contains (its tracking ref before the fetch, or the merge base). An entry only local Bolt holds is never ended; state that in D6 and add an acceptance criterion for it.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4 live bullet, D7, D8 wake bullet
    summary: At wake every paused run's driver strand is still live and its loop lock held, so D4 judges each pair live and never moves it, D7 has no live input and falls to active, and D8 lifts the hub pause onto stale wefts, which is the Neve incident the task fixes
    fix: In D4, state that under a live hub pause entry a paused pair is not live for the reconcile, so its branches are judged and moved like any quiet pair. In D7, map a live pair whose branch is behind or diverged to stopped, so D8 gives it a pair pause entry before the hub lift. Add an acceptance criterion - wake with a paused run whose weft moved on origin fast-forwards it, or pair-pauses it when it cannot.
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 first paragraph, D9, D10 records-side bullet, D6 Finalize-entered exception
    summary: D5's set-aside needs D3's ended set, which only the reconcile computes, so Fabric.Pull and the push retry either replay an ended run's stale commits or need an unstated Bolt read, and if they read origin's Bolt they set aside this machine's own landing records, since only D6 excludes a run with Finalize entered
    fix: In D5, make the ended set an argument of the helper and state its source per caller - the reconcile passes D3's set; Fabric.Pull and the push retry pass the set read from the already-fetched origin Bolt tracking ref with no extra network call, or pass none and set nothing aside (pick one and say so). In every case apply D6's Finalize-entered exclusion inside D5, so a run landing on this machine is never set aside. Add an acceptance criterion - a prime-weft push rejected right after this machine's Finalize pushed done keeps the landing's commits.
  - id: F5
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D8 lyx cleanup bullet
    summary: lyx cleanup without --apply is a read-only dry run that changes nothing, yet D8 runs the hub reconcile before the segments in both modes, so a dry run fast-forwards branches, replays records and tears down ended pairs
    fix: In D8's cleanup bullet, state that without --apply the reconcile runs judge-only (both ls-remote calls and the fetch, no move, no replay, no teardown) and its report is printed, and that only --apply runs the moving reconcile; a D3 stop refuses both modes. Update cleanupcli's bound accordingly and add an acceptance criterion for the dry run.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 prime bullet, D8 orch start, wake and cleanup
    summary: Batten's prime lock wait is ten minutes and parentlock.Acquire has no limit, the parent lock is held from Merge-In to Finalize and across a sleep, and its queue grants only run owners, so orch start, wake and cleanup can block minutes or without bound before reporting busy
    fix: In D4, replace each with batten's bounded wait by one try-acquire of the prime lock and one read of the prime branch's parent lock record (held by a live owner means busy), with no wait and no ticket; state that the reconcile takes no parent lock ticket, since it is no run, and that a held lock reports busy at once.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1, D4 live bullet, D6 step 1, D10 notice
    summary: pairteardown, reedengine, creelcli and orchcli all import fabricengine, so a fabricengine entry point cannot tear down through pairteardown, read driver-strand liveness or send creel mail, and the record names no seam for any of them
    fix: In D1, state that fabricengine owns the git judgment, moves and replays, and that the liveness probe, the ended-entry teardown and the push-failure notice are seams the caller injects (a live-pair probe, a teardown func over pairteardown's local mode, a notify func over orchcli.NotifyPrime), wired by each call site's CLI package and by fabriccli for the detached push child.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10 notice paragraph, D8 orch start bullet
    summary: NotifyPrime posts nothing when no orch strand is recorded and never retries, yet the exclusive-created record suppresses every later mail, so a 403 episode that starts while no orch runs is never reported to any orch
    fix: In D10, keep a notified flag in the failed-push record, set only once the mail was posted, and have a later failure of the same branch mail while it is unset. In D8's orch start bullet, add the outstanding failed-push records to the printed report and the start note.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5 set-aside tags, D6 step 1, Auto-mode assumptions archive tags stay local
    summary: Bolt and every weft share one records repo, and reportengine reads every local archive tag as a run archive, so each set-aside Bolt or weft commit and D6's local weft-tip tag add phantom or duplicate runs to lyx report and skew its per-slug chain
    fix: In D5, tag set-aside commits under a separate local prefix such as set-aside/<slug>/<sha>, which no archive reader matches. In D6, either keep the archive/<slug>/<sha> tip tag and state that reportengine dedupes it against origin's tag of the same tip, or use the same set-aside prefix; pick one and say so.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10 gone bullet
    summary: Gone can only come from the local tracking ref, which only the last fetch refreshes, and the detached push child fetches nothing, so a parked pending push respawned at wake before the reconcile recreates a branch deleted on origin
    fix: In the gone bullet, state the source - every records-side push first runs one ls-remote of that branch, or pushes only after a reconcile or fetch that pruned the tracking ref - and state its cost. Also state that a pending flag respawned at wake waits for the wake reconcile.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D10 records-side bullet (deferral)
    summary: The push holder takes the owed flags before each step, so for the flag to stay set a deferral must rewrite it, which sends the holder straight back into a push-reject-defer loop for as long as the live run's status.json stays dirty, and leaving the flag unset makes the rejection silent
    fix: In D10, state that a deferral leaves the commits to the next commit's push and ends the holder's loop - it writes no flag of its own side - and writes the failed-push record with class deferred and no mail, which the next landed push removes. Name the bound - at most one rejected push per commit.
  - id: F12
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 first paragraph, D8 batten resume bullet, Acceptance criteria resume refused
    summary: D6 says the run scope never applies this decision to its own run, but batten resume is run scope over its own run, so its ended refusal and the acceptance criterion resume refused naming the landing can never fire
    fix: In D6, change the sentence to say the run scope classifies its own run as ended like any other but never runs D6's teardown steps on it.
  - id: F13
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 busy paragraph, D7, D8 wake bullet
    summary: An ended entry whose driver is busy is reported busy, which D7 files under stopped, so at wake it gets a pair pause whose notice names lyx resume <slug>, re-driving a landed run D6 says is never resumed
    fix: In D7, state that ended takes precedence over every stopped cause, so an ended-and-busy entry is ended with a busy flag. In D8's wake bullet, give such an entry a pair pause entry whose notice names the landing or removal and the teardown way forward, never lyx resume <slug>.
  - id: F14
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 ahead and diverged bullets
    summary: Fabric.Pull recovers a rewritten warp upstream with no unpushed commits by its anchor walk and reset, and ResetPairCode rewinds a task branch on origin under a lease, but D4 stops the first as diverged and reads the second as ahead, so the next push restores abandoned commits
    fix: In D4, state that a code branch whose upstream was rewritten and that holds no unpushed commit is left to Fabric.Pull's existing recovery and reported as rewritten, not diverged, and that a code branch whose origin head is an ancestor reached by a rewind is reported as ahead with a warning naming the rewind, never pushed by the reconcile.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Constraints, PATTERN-fabric-destruction-chokepoint line
    summary: The constraint routes the local tags through destroy.go's executors, but archive.go states tag creation is not destructive and lives outside destroy.go
    fix: Drop the local tags from that constraint line, keeping only the replay's abort and restore.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D8 Merge-In and Finalize bullet
    summary: Finalize already holds the parent lock it took at Merge-In, which Merge-In keeps on every way out, so before taking the parent lock misdescribes Finalize's order
    fix: Say Merge-In runs the run-scope reconcile before taking the parent lock, and Finalize runs it at entry under the lock the run already holds.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D9 bound
    summary: The bound that the pull never discards a non-empty local records commit ignores D5's set-aside, which moves an ended entry's non-empty commits off the branch
    fix: Append to D9's bound - except an ended entry's commits D5 sets aside, which stay under their local tag.
  - id: F18
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10 notice paragraph
    summary: A synchronous push run from an agent pane, such as lyx loom commit-records, reaches the orch signed as that strand, since NotifyPrime posts as the strand first and creel refuses a program claim from a non-orch pane
    fix: Reword to - signed as the program FABRIC, or as the calling strand when the push runs in an agent pane, as NotifyPrime does.
  - id: F19
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 records lock bullet, D10 records-side bullet
    summary: Bolt has two push locks, board.push.lock held by Bolt.Sync and .gitrepo-push.lock held by pullFirst and the seed drop, and the record names neither for the Bolt replay or the Bolt retry
    fix: In D4, name the Bolt order as board.lock then .gitrepo-push.lock, and in D10 state that a Bolt push retry holding board.push.lock try-acquires board.lock and then takes .gitrepo-push.lock.
  - id: F20
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D10 gone bullet, D6
    summary: A parked push of an ended entry's weft that origin's cleanup deleted is classed gone and mails the orch, though D6 treats the same state as a routine ended entry
    fix: In D10, state that a gone branch of an entry in the ended set writes no failed-push record and is left to D6's teardown.
  - id: F21
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/support-log.md Question ledger Q8
    summary: Q8 records the wake pair pause entry as owned by the request, while D8 and Q20 say it is not owned by the request
    fix: Change Q8's answer to - a plain pair pause entry, not owned by the request, lifted with lyx resume <slug> (superseded by Q20).
  - id: F22
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Goal last sentence
    summary: The Goal cites D8 for the unreachable-origin exception, but that exception lives in Auto-mode assumptions, not D8
    fix: Change an unreachable origin (D8) to an unreachable origin (Auto-mode assumptions).
---

### [BLOCKING] Bolt replay rests on a false premise about Bolt's layout

**Location:** `_lyx/discussion/decision-record.md` D5, D3, Acceptance criteria (Neve's case)

**Issue:** `internal/boardengine/doc.go:7` holds every entry, status included, in one `board.json`.
`doc.go:82` re-renders `README.md` on every write, opening with `Rendered <minute> UTC` (`_board/README.md:3`).
`git -C _board log --stat` shows every `board sync` commit changing `README.md`, `board.json` and `design-<slug>.md`, and `_board` has no merge driver.
So "a commit whose every changed entry file is an ended entry's" has no file to test.
A cherry-pick of any kept local Bolt commit onto a moved origin conflicts on `README.md`, so D3 stops every scoped entry after nearly any two-machine Bolt divergence, and Neve's acceptance case holds only when every local commit is set aside.

### [BLOCKING] An entry origin never held is classed ended and torn down

**Location:** `_lyx/discussion/decision-record.md` D3 step 2, D6

**Issue:** "Gone" is read as absent from origin's Bolt tip.
An entry created while Bolt pushes failed, as in the three-day 403 of #737, is absent from origin without ever having been removed.
With a local batten run record it is ended, and pairteardown's quiet rule (`internal/pairteardown/doc.go:52`) lets a parked driver be ended, so a live run is removed and its resume refused.

### [BLOCKING] Wake never moves or stops a paused run's pair

**Location:** `_lyx/discussion/decision-record.md` D4 live bullet, D7, D8 wake bullet

**Issue:** Under the hub pause a driver strand is interrupted but live, so D4 judges each pair live and never moves it.
D7 has no live input, so the entry is `active`, D8 adds no pair pause, and the hub lift resumes the driver on a weft behind origin.
That is the incident the Goal names, caught only later by a push conflict.

### [BLOCKING] D5's ended set is undefined outside the reconcile and can catch this machine's own landing

**Location:** `_lyx/discussion/decision-record.md` D5, D9, D10, D6

**Issue:** Only the reconcile builds D3's ended set, but `Fabric.Pull` and the push retry also call D5.
Without the set they replay stale ended commits, and the Turn 6 claim that D5 sets them aside everywhere does not hold.
With a set read from origin's Bolt, the Finalize case breaks: this machine pushes Bolt `done`, its prime-weft push is rejected, and the retry sets aside the landing's own status commits, since only D6 carries the Finalize-entered exclusion.

### [BLOCKING] A cleanup dry run would mutate the hub

**Location:** `_lyx/discussion/decision-record.md` D8 `lyx cleanup` bullet

**Issue:** `internal/cleanupcli/doc.go:6` and `lyx batten cleanup --help` define the run without `--apply` as read-only.
D8 runs the moving hub reconcile, D6 teardown included, before the segments in both modes.

### [MEDIUM] The prime's lock waits are not bounded at hub call sites

**Location:** `_lyx/discussion/decision-record.md` D4, D8

**Issue:** `internal/battenshed/primelockwait.go:19` waits ten minutes, and `internal/parentlock/doc.go:29` says Acquire has no time limit.
The parent lock is kept from Merge-In through Finalize (`internal/landingshed/mergein.go:114`) and across a sleep (`parentlock/doc.go:15`), and its queue grants run owners only.

### [MEDIUM] The reconcile's teardown, liveness and notice steps cannot live in fabricengine

**Location:** `_lyx/discussion/decision-record.md` D1, D4, D6, D10

**Issue:** `go list` shows pairteardown importing fabricengine, `pairteardown/doc.go:4` records reedengine importing it, and creelcli and orchcli depend on it.
The record names no seam for these steps, so D1's single fabricengine entry point cannot do what D4, D6 and D10 ask of it.

### [MEDIUM] The single notice is lost when no orch is recorded

**Location:** `_lyx/discussion/decision-record.md` D10, D8

**Issue:** `internal/orchcli/notify.go:99-101` logs and posts nothing without an orch strand, and a failed post is not retried.
After that, the exclusive-created record suppresses every later mail, and orch start does not print outstanding records.

### [MEDIUM] Set-aside tags collide with the archive namespace reportengine reads

**Location:** `_lyx/discussion/decision-record.md` D5, D6, Auto-mode assumptions

**Issue:** `internal/reportengine/source.go:76` lists every local `archive/` tag of the records repo, which Bolt shares with the wefts.
`runs.go:129-150` turns each tag into a run or a finding and chains it as the slug's previous tag.

### [MEDIUM] The gone class has no source fresher than the last fetch

**Location:** `_lyx/discussion/decision-record.md` D10 gone bullet

**Issue:** The detached push child fetches nothing (`internal/fabricengine/coalesce.go:144-187`), and a plain `git push` recreates a deleted branch.
A pending flag respawned at wake can push before the wake reconcile has fetched.

### [MEDIUM] The push deferral either loops or goes silent

**Location:** `_lyx/discussion/decision-record.md` D10 records-side bullet

**Issue:** `internal/fabricengine/pendingflags.go:116-137` takes the owed flags before each step and sends a holder whose flag is set again back to the lock.
A deferral that keeps the flag set loops push, reject and defer while the live run's `status.json` stays dirty.
One that leaves the flag unset leaves the rejected commits with no record and no mail.

### [MEDIUM] The own-run exclusion makes resume's ended refusal unreachable

**Location:** `_lyx/discussion/decision-record.md` D6, D8, Acceptance criteria

**Issue:** Batten resume is run scope over its own run, and D6 never applies itself to that run, so no resume ever sees `ended`.

### [MEDIUM] An ended entry with a busy driver gets resume as its way forward

**Location:** `_lyx/discussion/decision-record.md` D6, D7, D8 wake bullet

**Issue:** Busy falls under `stopped` in D7, so at wake the entry gets a pair pause, and its notice names `lyx resume <slug>` for a run that landed elsewhere.

### [MEDIUM] Code-branch outcomes misread existing rewrites

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** `internal/fabricengine/pull.go:296-330` recovers a rewritten warp upstream that has no unpushed commits.
`destroy.go:1822-1829` (`ResetPairCode`) rewinds a task branch on origin under a lease.
D4 stops the first case as diverged with no way forward, and reads the second as ahead for the next push to carry.

### [LOW] The destruction-chokepoint constraint covers tag creation

**Location:** `_lyx/discussion/decision-record.md` Constraints

**Issue:** `internal/fabricengine/archive.go:2` says tag creation is not destructive and lives outside destroy.go.

### [LOW] Finalize does not take the parent lock fresh

**Location:** `_lyx/discussion/decision-record.md` D8

**Issue:** `internal/landingshed/mergein.go:114` keeps the lock on every way out, and `finalize.go:147` runs under it.

### [LOW] D9's bound ignores the set-aside

**Location:** `_lyx/discussion/decision-record.md` D9

**Issue:** D5, which D9 calls, removes an ended entry's non-empty commits from the branch.

### [LOW] FABRIC is not always the signer

**Location:** `_lyx/discussion/decision-record.md` D10

**Issue:** `internal/orchcli/notify.go:104-113` posts as the strand first, and `internal/creelengine/identity.go:64-66` refuses a program claim from a non-orch pane.

### [LOW] Bolt's two push locks are not named

**Location:** `_lyx/discussion/decision-record.md` D4, D10

**Issue:** `internal/fabricengine/bolt.go:130` locks `board.push.lock` and `bolt.go:236` locks `.gitrepo-push.lock`.
Which of them "the push lock" means on Bolt decides whether the order holds.

### [LOW] A gone ended weft raises a false alarm

**Location:** `_lyx/discussion/decision-record.md` D10, D6

**Issue:** The landing machine's cleanup deletes the weft on origin, so the parked push is classed gone and mails, although D6 treats the same state as routine.

### [LOW] Q8 contradicts D8

**Location:** `_lyx/discussion/support-log.md` Question ledger

**Issue:** Q8 says the pair pause entry is owned by the request, while D8 and Q20 say it is not.

### [NIT] The Goal cites the wrong section

**Location:** `_lyx/discussion/decision-record.md` Goal

**Issue:** D8 does not mention an unreachable origin; Auto-mode assumptions does.

## Rejected

- No Decision changes this task's own board entry: D6 step 2 departs from the entry's local run-record removal in design only, and D11 changes Preflight code, not the entry.
- Relocation check: the Rationale lines of D1, D5, D10 and D11 each carry a premise the plan writer acts on, and Open risks names residual risks, not deliberation, so nothing in the decision record is moved out.
- The lock order of D4's write-then-push against D10's try-acquire held under every interleaving tried, so it is no finding beyond F19's naming gap.
- The D11 detached Preflight push held: D10 makes a refused detached push visible, which is the reason D11 gives.
