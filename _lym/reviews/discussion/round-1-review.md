---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:93
    summary: "With origin unreachable a local landing cannot return done: ownerChecks runs the run-scope reconcile before any merge (finalize.go:359, 439), which D6 turns into a git-transport wait; syncParent's ReadParentTips(true) fetches first (finalize.go:612); handedOverStop reads origin before each push (finalize.go:957, 987, 993). The Goal, D5 and the D5 criterion 'returns done while origin is unreachable' contradict D6's criterion that the owner check on an unreachable origin is a git-transport stop."
    fix: "In D5, settle what each origin read of a local landing does while origin is unreachable: the owner checks' reconcile in gateUnlocked and landUnderLock, syncParent's fetched tips read, and handedOverStop before the parent, records and done steps. Either state that a local landing waits (D6) until origin answers for its owner verdict, which PATTERN-owner-machine requires, and rewrite the Goal's landing sentence and the D5 criterion to say it lands once origin answers, or name the local reads that replace each origin read and the PATTERN-owner-machine departure that allows it. Make the D5 and D6 criteria agree."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:95
    summary: "D5 covers a local parent that lags or has diverged, but not one ahead of origin. After landing A's parent push is owed, landing B on the same parent stops Stuck in syncParent's ParentAhead case, 'ahead of its upstream; way forward: push or reconcile' (finalize.go:619-620), so local landings serialize on the network, against D5's 'other runs keep committing' and the Goal."
    fix: "Add to D5 a rule for a local parent ahead of origin: when every commit it is ahead by is a tip the landing target records as owed, Finalize lands on it as on an equal parent, and the landing target then owes the newer tip; any other ahead commit keeps today's Stuck reason. Add a D5 criterion: a second local landing on a parent whose first landing push is owed merges and records the newer owed tip."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:98
    summary: "Nothing holds the board done back. markTaskDone is a board write (finalize.go:996), every board write spawns the board sync (boardengine/board.go:216-218), and D1 hands that sync to the board's own target, which pushes every owed board commit. So done can reach origin before the landing target's parent push, which breaks D5's ordering promise. D1's list of kept synchronous pushes does not name the done write either."
    fix: "In D5, name the mechanism that orders done after the parent push: either Finalize records the done write as owed on the landing target and the board write that carries it takes no board-target wake, and the board target skips pushing past a done commit whose landing target still owes its parent; or the done write is made only by the landing target once its parent push landed. Pick one, state it in D5, and say in D1 which board target rule it changes."
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:73
    summary: "A refused target leaves the ledger, and no pusher or dobby ahead check writes it back while its refused-push record stands (runpusher/start.go:122-125, run.go:66-68). The ledger can empty while a refused branch is still only local: D3 posts 'every branch is pushed' and D4 shows sync 0m, the state the operator reads before a machine switch. The record does not say what clears the refusal or what the bar shows meanwhile."
    fix: "In D3, keep a refused target's entry in the ledger, marked refused, until its refused-push record is cleared, so the all-clear never posts and the bar's minutes keep counting while it stands. State that clearing the record through its way forward removes the mark and lets the pusher start again, and that the dobby's ahead check writes no new entry for a target whose refused mark stands. Add a D3 criterion that a refused target keeps the bar's minutes counting and blocks the all-clear."
  - id: F5
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:112
    summary: "'While it waits the driver holds its run lock' cannot work. The step takes shedrun.RunLock (loomcli/wiring.go:400), the driver's recheck try-acquires it after each wait and exits as running-elsewhere when it is held (sheddriver/drive.go:854-862, battencli/wire.go:193-204), and teardown and cleanup wait on or try it (pairteardown/teardown.go:995, battencli/cleanup.go:220). The driver's own lock is shedrun.DriverLock (battencli/drive.go:72)."
    fix: "In D6, replace 'run lock' with the driver lock, shedrun.DriverLock, in the wait-record rules: a wait record under a held driver lock reads as waiting, and one whose driver lock is free is a dead driver. Change the D6 criterion to match, and state that the driver holds no run lock between steps."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:96
    summary: "D5 leaves the crash and resume gap between parentHandle.Merge and the owed record open. A Finalize killed after the local merge and before the landing entry is written leaves no owed tip. On resume, ParentHoldsTaskWork reads the local parent and skips the merge, and the run reader reads origin's parent, which lacks the work (runread.go:273-280). The parent push is then never owed and the run never reads as past its merge."
    fix: "In D5, state that the landing entry is recorded on every Finalize pass that finds the local parent holding the task's work while origin's parent lacks it, the resume included, and that the run reader's past-the-merge read checks the local parent whenever origin's parent lacks the work and the local parent holds it. Add the resume after a crash between the merge and the record to the D5 resume criterion."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:97
    summary: "Finalize's records commit goes through the pair's pusher, which pushes on its own schedule, so origin's run record can read Finalize done while origin's parent lacks the work. runReader reads childOnOrigin and originParentHolds for a reader whose pair is absent (runread.go:129, 222-224, 269-285). D5 orders only the board done after the parent."
    fix: "In D5, state the order of the run's records commit against the parent push: the landing target pushes the parent before the pair's pusher may push the records commit Finalize makes, or the pair target holds that commit while its landing target owes a parent push. Add the case to the D5 ordering criterion."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:184
    summary: "'Owes a landing push' is read by D5's past-the-merge read, D5's pull-request check and D13's landed-pair teardown, but the record never says whether it means a ledger entry or a parent ahead of origin. With D3's 'a refused target leaves the ledger', a refused landing push reads as not owed: the run reader switches to origin's parent, D13 stops waiting, and D5's pull-request check sees no owed push where D5 says a refused one is a blocked halt."
    fix: "In D5, define 'the landing target owes a parent push' once, as the landing target's ledger entry standing, refused or not (with F4's refused mark), and have D5's three reads and D13's teardown read that definition. State that a refused landing push makes the pull-request check a blocked halt naming the refusal's way forward."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:75
    summary: "D3 posts its summary only once some ledger target has stayed owed past the threshold, so a run D6 holds waiting on a GitHub API outage with nothing owed never gets a summary. D6 says the waiting run 'shows only in D3's summary', and its bound relies on that summary, and the Auto-mode assumption says the threshold also governs a waiting run."
    fix: "In D3, add the second trigger: the dobby posts the summary once some target has stayed owed, or some run has kept a wait record, for fabric.push_notice_after_min. Count the all-clear's owed period from the first of either. Add a waiting run with an empty ledger to the D3 summary criterion."
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:182
    summary: "With git or pushes off, origin's tracking refs never hold the tips, so under D13 Worktree-Teardown always leaves the pair standing and the dobby never removes it. That contradicts D11's criterion and PATTERN-pair-remote's background, both of which say that with pushes off the teardown pushes nothing and removes the pair."
    fix: "In D13, add that with git or pushes off Worktree-Teardown sets no mark, reads no tracking ref and removes the pair as today."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:43
    summary: "D1 sorts push paths by naming callers, and the list is wrong both ways. The entry names lym fabric commit's detached push (fabricengine/commit.go:171), which D1 leaves out. tendercli/deps.go's board pushes, which D1 keeps as synchronous because 'a caller reads its result', are detached spawns (deps.go:148, 154). heddlecli/cli.go:173, pairteardown/followup.go:91, configcli/hubwide.go's bolt.Push, the catch-up's pushOwedForCatchUp and PR-Review's PushRun calls (loomcli/landingdeps.go:423, 473, 905) fall under neither list."
    fix: "Rewrite D1's rule by kind: every detached push spawn, SpawnDetachedPush, SpawnStatusPush and boardengine.SpawnSync, starts or wakes the pusher for its target instead. Every synchronous push keeps its push and is listed by the reason its caller reads the result: the board owner write, hubreconcile, config hubwide, webster reset, the catch-up (D8) and PR-Review's task-branch push, which GitHub needs on origin. Drop tendercli from the synchronous list."
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:57
    summary: "D2 does not say that the pusher's existing retry loops yield to the interval in force. Run still calls PushUntilLanded, which retries from 1 s doubling to 60 s (runpusher/run.go:144, pushuntillanded.go:22-39), and pushClassified retries a transport failure over its own backoff (pushclassified.go:192-206). So 'at most once per interval in force' does not hold."
    fix: "In D2, state that the pusher's owed retry is the interval in force alone: PushUntilLanded's one-second-to-a-minute loop and its ceiling notice are replaced by one push per interval, and the pusher's pushes run pushClassified with no transport backoff of their own. Add the no-extra-attempt case to the D2 criterion."
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:112
    summary: "D6's 'it writes no halt note' misses where the note is written: loomAfterStep writes it in the step process for a failed halt with a transient class (loomcli/halt.go:2, 37-38, 127), before the driver reads the envelope or writes a wait record. Every waited stop would still leave a halt note."
    fix: "In D6, state that the step writes no halt note for a failed stop whose transient class is git-transport or github-api, and that the driver writes one only when it stops waiting on a stop that is no longer network-class. Change the D6 halt-note criterion to say the step writes none."
  - id: F14
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:42
    summary: "A landing onto a beam parent gets two pushers on one branch. D1 makes the beam pair's target owe either branch, its code branch included, and also adds a landing target for that same parent branch. The pusher lock is per records worktree (runpusher/target.go:48-51), so the two either share the beam pair's lock or race each other unordered, and D3 lists the branch twice."
    fix: "In D1, state that a landing target exists only for a parent branch no pair target serves, the prime's code branch; a landing onto a beam parent records its owed tip and push-now mark on the beam pair's target, whose pusher pushes the beam's records side, then its code branch, then the board done (F3)."
  - id: F15
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:148
    summary: "D9 rests on an unpinned and unsettled git fact: git cannot evaluate ignore rules read from a tree. git check-ignore and git ls-files read per-directory ignore files from the worktree, and the only tree option is --with-tree, which does not read ignore files from the tree (git check-ignore -h, git 2.53.0). lym would have to read the ignore blobs and match them itself, so nested ignore files, negations, info/exclude, core.excludesFile, symlinks and nested repositories are left unsettled."
    fix: "In D9, record how the first parent's rules are evaluated, with its pin: for example, materialize the first parent's ignore files for the deleted directory's path chain into a temporary index-free check with git check-ignore --no-index -X per file, or match them in Go. State which rule sources count (per-directory ignore files from the first parent, never info/exclude or core.excludesFile), state that a symlink or nested repository under the directory is never removed, and cite the git documentation or a recorded command and its output for the behaviour relied on."
  - id: F16
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:147
    summary: "D9's placement and base misfire on a resume and a fast-forward. On a resume, mergeParent runs clean 'before the merge-in' before any merge (finalize.go:1344), so the leftovers halt there before D9's removal runs. The 'merge commit's first parent' is not the pre-merge task tip when the merge-in fast-forwards, since MergeCommit is the task head after the merge (finalize.go:1369-1373). The merge record keeps that pre-merge TaskHead (finalize.go:1382)."
    fix: "In D9, take the base from the merge record's pre-merge task head, never the merge commit's first parent. Run the removal before the before-the-merge-in clean check too, whenever a merge record names a base whose tree holds the deleted directory. Add the resume and fast-forward cases to the D9 criterion."
  - id: F17
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:129
    summary: "D7's reachability bound rests on unpinned git behaviour and has no way forward. A hand commit can enter the local code branch's reflog without the branch keeping it (fetch, reset to origin, reset back), so it passes and is reset away. lym's own rewound commits can leave the reflog after expiry (gc.reflogExpireUnreachable, 30 days by default), so the reset is refused. The refusal's notice names the commit but no way forward, and the takeover stop is accepted with none."
    fix: "In D7, record the git behaviour relied on with its pin, the reflog expiry defaults from git-gc's documentation and a test in internal/fabricengine that drives a reflog-held commit, and state the bound's known gap. Give the refusal a way forward the notice names, such as merging origin's branch into the task branch through the task worktree and running lym fabric push, and say the same for the takeover stop."
  - id: F18
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:246
    summary: "The declared PATTERN-pair-remote text, 'Pushing anything lym owns ... one background pusher per target ... which no transition waits for', contradicts D1's kept synchronous pushes (the board owner write, hubreconcile, webster reset, whose owed record holds the run at its transition) and D8's synchronous catch-up push. Those paths are not exempted in the entry text. The board and landing targets also have no records and code sides, so 'records side before code' does not apply to them."
    fix: "Rewrite the PATTERN-pair-remote change text so it binds the pushes D1 moves and names the exception: for example 'Pushing lym's branches: a detached push only through one background pusher per target, records side before code where a target has both, retrying an owed push quietly; a push whose caller reads its result stays synchronous; ...', kept under the rule-text cap."
  - id: F19
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:221
    summary: "The bug-entry rule asks, for each observed fault, for a test that fails on the merge base, a reason no test seam exists, or a statement that no bug is fixed. Issue #238, the first observed fault in Goal, has none: its D1 and D3 criteria are not stated red-first and do not say where the merge base fails."
    fix: "Make one D1 or D3 criterion red-first for issue #238: a test in internal/fabricengine (tier1) where an owed failure of lym fabric sync's push mails the orch on the merge base and, after the change, starts the pusher, writes no failed-push record and posts no notice."
  - id: F20
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:119
    summary: "D6's bound relies on the network class, but some permanent faults read as network-class: an origin URL whose host never resolves ('could not resolve host' in gitexec's transport phrases) and a GitHub API path refused at every connection (githubclient.IsTransient is true for any non-parse url.Error, transient.go:43-47). The record does not decide whether each is re-stepped forever. Rate limits, 403 and 404 already read as not transient (transient.go:28-41)."
    fix: "In D6's bound, state that a misconfigured host or proxy that never answers is re-stepped like an outage and surfaces only in D3's summary and lym batten cancel. Also name the classes that already leave the rule at once: GitHub rate limit, 401, 403 and 404."
  - id: F21
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:128
    summary: "D7 states as new work a classification that already holds: authRefusalTriggers already holds 'Authentication failed' and 'could not read Username' (gitrepo/push.go:58), rejectionRetryKind treats ErrPushAuthRefused as final (pushclassified.go:345), and a GitHub 401 is not transient (githubclient/transient.go:40-41)."
    fix: "In D7, say that the classifier already reads these as refusals and that this task keeps that, and turn the D7 authentication criterion into a check that the pusher stops at once with one record and one notice on such a refusal."
  - id: F22
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:132
    summary: "D7's 'the same rule binds every other push path that resets or forces' binds paths only by reference to pr-modes-no-agent's list. None of the five paths that list names forces a push: they reach the reset only through resetUnnamedTip. The other updateRemoteBranch callers, fabricengine/handover.go:274 and destroy.go:2697, are left unsettled."
    fix: "In D7, state that the rule lives in resetUnnamedTip, which the listed paths reach, and say for each other updateRemoteBranch caller (the hand-over and destroy moves of origin branches) whether the reachability rule binds it and why."
  - id: F23
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:140
    summary: "The catch-up pushes the prime's records before the pair (battencli/catchup.go:265-275). An owed failure of the prime's push belongs to the prime's target, not to 'that run's pusher', and D8 does not say which pusher starts."
    fix: "In D8, state that an owed failure of the prime's records push starts the prime's pusher and an owed failure of the pair push starts the pair's pusher, and that the run starts in either case."
  - id: F24
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:17
    summary: "The hand-commit reproduction does not reproduce the fault: resetUnnamedTip runs only on a rejected push, which needs a local commit origin lacks. After 'add a commit on GitHub, then push the pair' with nothing local ahead, there is nothing to push."
    fix: "Change the reproduction to: add a commit on GitHub to a task's code branch, commit on the local code branch, then push the pair."
  - id: F25
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:151
    summary: "D9's bound deletes every file the first parent's rules ignore under the deleted directory, but ignored is not the same as build output: an ignored local settings file, a .env or a developer's notes in the deleted project are removed too. The rationale claims all such files are build output, and the bound does not say what it lets through."
    fix: "In D9's bound, state that any file the first parent's rules ignore is removed, local settings and secrets included, that each removal is logged with its path, and that this is accepted because the project is gone from the parent."
  - id: F26
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:215
    summary: "Open risk 1 (hub versus machine for the sync field and the all-clear) is a choice the writer can make and already made (Q3, Auto-mode assumption 1), not a question only the parent can settle. A choice the writer can make is never routed to the parent."
    fix: "Drop Open risk 1 and keep the hub pick as the Auto-mode assumption, with the reason that each hub has its own origin and orch and that the entry's 'this machine' is read as this machine's hub."
  - id: F27
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:54
    summary: "D2 says 'Two keys in internal/fabricengine/template.yaml' and 'The keys are new', while D3 adds a third new key, fabric.push_notice_after_min, without saying where it is declared."
    fix: "In D2, name all three keys, fabric.push_interval_s, fabric.push_ceiling_s and fabric.push_notice_after_min, as declared in internal/fabricengine/template.yaml, and have D3 refer to D2 for the key."
---

### [BLOCKING] A local landing cannot return done while origin is unreachable

**Location:** `_lym/discussion/decision-record.md:93` (D5, with D6 and the D5 criterion)

**Issue:** Before a local landing merges, Finalize runs its owner checks twice, and both run the run-scope reconcile against origin (`internal/landingshed/finalize.go:359`, `:439`; `finalize_owner.go:47-55`).
D6 turns the unreachable result into a `git-transport` wait.
`syncParent` reads the parent tips with a fetch (`finalize.go:612`), and `handedOverStop` reads origin before the parent push, the records push and the done write (`finalize.go:957`, `987`, `993`).
So with origin unreachable the landing waits; it does not merge and return done.
PATTERN-owner-machine requires a landing to be judged against origin, so the waiting behaviour may be the right one.
Either way, the Goal, D5 and the D5 criterion claim the opposite of D6's criterion. All four lenses found this.

### [BLOCKING] A second local landing halts on a parent ahead of origin

**Location:** `_lym/discussion/decision-record.md:95` (D5)

**Issue:** Once one landing's parent push is owed, the local parent is ahead of origin, and `syncParent` returns Stuck for `ParentAhead` (`finalize.go:619-620`).
D5 handles only the lagging and diverged cases, so every later landing on that parent stops until the first push lands.

### [BLOCKING] The board done is pushed by the board target, not after the parent

**Location:** `_lym/discussion/decision-record.md:98` (D5, with D1)

**Issue:** `markTaskDone` is a board write (`finalize.go:996`), and every board write spawns the board sync (`internal/boardengine/board.go:216-218`).
Under D1 that sync wakes the board target, which pushes every owed board commit.
Nothing holds the done commit back, so D5's ordering promise has no mechanism.

### [BLOCKING] A refused target leaves the ledger and the hub reads as pushed

**Location:** `_lym/discussion/decision-record.md:73` (D3, with D4 and D7)

**Issue:** A refused target leaves the ledger.
No pusher starts while its refused-push record stands (`internal/runpusher/start.go:122-125`, `run.go:66-68`), so the dobby's ahead check never writes the entry back.
The all-clear then posts and `sync:` reads 0m while the refused branch is still only local, which defeats D4's purpose.

### [BLOCKING] D6's wait-record rule names the run lock, which the driver cannot hold between steps

**Location:** `_lym/discussion/decision-record.md:112` (D6)

**Issue:** The step takes `shedrun.RunLock` (`internal/loomcli/wiring.go:400`).
After every wait, the driver's `recheck` try-acquires it and exits as running-elsewhere when it is held (`internal/sheddriver/drive.go:854-862`).
Teardown and cleanup wait on it or try it.
The driver's own lock is `shedrun.DriverLock` (`internal/battencli/drive.go:72`).

### [MEDIUM] D5 leaves the crash gap between the merge and the owed record

**Location:** `_lym/discussion/decision-record.md:96`

**Issue:** If Finalize is killed after the local merge and before the owed record, the resume skips the merge and records no owed tip.
The run reader then reads origin's parent (`internal/pairteardown/runread.go:273-280`), which lacks the work.

### [MEDIUM] The records commit can reach origin before the parent

**Location:** `_lym/discussion/decision-record.md:97`

**Issue:** The pair's pusher pushes Finalize's records commit on its own schedule.
Origin's run status can then read done while origin's parent lacks the work, and `childOnOrigin` and `originParentHolds` read exactly that pair of facts.

### [MEDIUM] "Owes a landing push" is undefined after a refusal

**Location:** `_lym/discussion/decision-record.md:184`

**Issue:** Three D5 reads and D13's teardown depend on whether the landing target owes a parent push.
Under D3 a refused push leaves the ledger, so all of them read "not owed".

### [MEDIUM] A run waiting on GitHub with nothing owed never enters a summary

**Location:** `_lym/discussion/decision-record.md:75`

**Issue:** D3's summary triggers only on a ledger target, while D6's bound and the Auto-mode assumption rely on the summary to show a long wait.

### [MEDIUM] D13 never removes a pair with pushes off

**Location:** `_lym/discussion/decision-record.md:182`

**Issue:** With pushes off the tracking refs never hold the tips, which contradicts D11's criterion and the PATTERN-pair-remote background.

### [MEDIUM] D1's list of push paths is wrong both ways

**Location:** `_lym/discussion/decision-record.md:43`

**Issue:** `lym fabric commit`'s detached push, which the entry names, is missing.
tender's detached spawns are listed as synchronous.
Several detached and synchronous push sites fall under neither list.
This is one finding about how D1 sorts the paths, not a list of missing call sites.

### [MEDIUM] D2's interval is undercut by the pusher's existing retry loops

**Location:** `_lym/discussion/decision-record.md:57`

**Issue:** `PushUntilLanded` and `pushClassified`'s transport backoff keep retrying inside one attempt, so a target is pushed more than once per interval in force.

### [MEDIUM] The step still writes a halt note for a waited stop

**Location:** `_lym/discussion/decision-record.md:112`

**Issue:** `loomAfterStep` writes the halt note in the step process before the driver decides to wait (`internal/loomcli/halt.go`).

### [MEDIUM] Two pushers serve one beam branch

**Location:** `_lym/discussion/decision-record.md:42`

**Issue:** A beam parent's code branch is owed both by the beam pair's target and by a new landing target, and each target has its own lock and order.

### [MEDIUM] D9's ignore evaluation from a tree has no git primitive

**Location:** `_lym/discussion/decision-record.md:148`

**Issue:** Git reads ignore files only from the worktree.
`git check-ignore -h` on git 2.53.0 offers no tree source, only `--with-tree`, which concerns index entries.
The record neither settles nor pins how the rules are matched.

### [MEDIUM] D9 misses the resume path and the fast-forward base

**Location:** `_lym/discussion/decision-record.md:147`

**Issue:** On a resume, the before-the-merge-in clean check (`finalize.go:1344`) halts before the removal runs.
In a fast-forward merge-in, the merge commit's first parent is the wrong base.

### [MEDIUM] D7's reachability bound is unpinned and its refusal has no way forward

**Location:** `_lym/discussion/decision-record.md:129`

**Issue:** The reflog can hold a hand commit the branch no longer keeps, and reflog expiry can drop lym's own commits.
The refusal names no way forward.

### [MEDIUM] The declared PATTERN text contradicts D1's synchronous exceptions

**Location:** `_lym/discussion/decision-record.md:246`

**Issue:** The new entry says "pushing anything lym owns" goes through one background pusher that no transition waits for.
D1 and D8 keep several synchronous pushes, and webster reset's owed record holds the run at its transition.

### [MEDIUM] Issue #238 has no red-first test

**Location:** `_lym/discussion/decision-record.md:221`

**Issue:** The bug-entry rule requires a test that fails on the merge base for each observed fault, and #238's criteria state none.

### [LOW] Permanent faults can be re-stepped forever as network-class

**Location:** `_lym/discussion/decision-record.md:119`

**Issue:** An origin host that never resolves and a proxy that refuses every connection both classify as transient, and the bound does not decide them.

### [LOW] D7's classifier sentence describes existing behaviour as new

**Location:** `_lym/discussion/decision-record.md:128`

**Issue:** Authentication failures and GitHub 401 are already refusals at the merge base.

### [LOW] D7 leaves the other origin-moving paths unsettled

**Location:** `_lym/discussion/decision-record.md:132`

**Issue:** The hand-over's and destroy's `updateRemoteBranch` calls are bound only by the phrase "every other push path".

### [LOW] D8 does not say which pusher an owed prime push starts

**Location:** `_lym/discussion/decision-record.md:140`

**Issue:** The catch-up pushes the prime's records first, and an owed failure there belongs to the prime's target.

### [LOW] The hand-commit reproduction needs a local commit

**Location:** `_lym/discussion/decision-record.md:17`

**Issue:** A pair push with nothing local ahead is never rejected, so `resetUnnamedTip` never runs.

### [LOW] D9's bound removes ignored files that are not build output

**Location:** `_lym/discussion/decision-record.md:151`

**Issue:** An ignored local settings file or secret under the deleted project is removed too, and the bound does not state this.

### [LOW] Open risk 1 is the writer's choice

**Location:** `_lym/discussion/decision-record.md:215`

**Issue:** Whether the field covers a hub or the machine was auto-picked (Q3) and is a design choice, not a question only the parent can settle.

### [NIT] D2 counts two keys while the record adds three

**Location:** `_lym/discussion/decision-record.md:54`

**Issue:** `fabric.push_notice_after_min` is a third new key that D2 does not name.

## Rejected

- existing-paths F1, "an owed push still writes a failed-push record": PATTERN-run-fsm's background ("The pusher retries it in its backoff and writes no owed record") and `RecordPushFailure` show that pusher pushes already write none, so D3's "still" is accurate. The detached pushes that do mail are the ones D1 moves to the pusher.

## Focus verdicts

- D3 with D1, D4 and D7, a refused target: today it leaves every signal, so the bar and the all-clear report it as pushed (F4). Only the record's way forward clears it.
- D5 with D6, a local landing with origin unreachable: the landing does not return done. It waits at the owner check, and the fast-forward's fetched tips read fails with a plain error (F1).
- D5 ordering and recovery: the crash gap (F6), the records and done order (F3, F7) and the refused landing push (F8) are open.
- D7 bound: covered by F17 and F22.
- D6 and D7 classification: rate limit, 403 and 404 already leave the rule. An unresolvable host and a refusing proxy do not (F20).
- D1 coverage: F11. Plain `git commit` and a beam merge are covered by D1's ahead rule and the dobby tick's ahead check.
- Attack surface: the push-now mark and `lym fabric sync-status` state their bounds. Removal, hand-over and cancel drop a ledger entry only once the worktree is gone, which loses no signal for a pair that stands.
- D9: F15, F16 and F25.
- Open risks: line 1 is the writer's choice (F26). Line 2 is a real conflict between the entry's "never stops other work" and PATTERN-owner-machine's catch-up rule, so it stays with the parent. Line 3 states a risk and asks nothing.
- Premises and board fit: the root-cause bullets match the code. D10's gate claim and D12's claim about #218 hold. `fabric-core-push-origin` depends on this task, and the Scope Out line respects that. `pr-modes-no-agent` landed as `34e64ac2e`, whose open point D7 answers.
- Acceptance criteria: F19 covers #238. Every other fault has a red-first test. D11 names its findings by reference to the entry.
