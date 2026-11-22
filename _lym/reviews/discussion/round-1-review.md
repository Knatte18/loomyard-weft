---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D5
    summary: "D5 and D8 rest on a false premise: the pusher pushes through `PushPairAnchored`, which itself writes owed-class failed-push records (Deferred, TeardownLock, Transport), and `ReadRun` turns those into `PushOwed`, so the act rule still holds the run"
    fix: "In D5's first bullet, state that the pusher's push path writes no owed-class failed-push record (Deferred, TeardownLock, Transport) and name the change in fabric that makes this so: `PushPairAnchored` gains a mode that returns the owed error without `recordOwedRecordsPush` or `teardownLockOwedError` writing a record, or the pusher's owed outcomes are written under a class `FailedPushesOwedBy` does not return. Cite the sites: `internal/fabricengine/pushanchored.go` (`recordOwedRecordsPush`, `teardownLockOwedError`), `internal/fabricengine/failedpush.go` (`FailedPushesOwedBy`), `internal/pairteardown/runread.go` (owed-push read). Rewrite D8's sentence on `runfsm`'s push-owed input to rest on that change, and state in D5 that the refused-push restart gate counts only refusal classes. Drop D3's bound clause that the pusher pushes only what `PushPairAnchored` already pushes, or qualify it with this change."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D3
    summary: "The prime's pusher serves every batten run but `PushRun` and `retryWeftPush` read a takeover only for one non-empty run slug, so a rejected prime push with no slug replays a taken-over batten run's stale commits onto origin instead of handing it over"
    fix: "In D3 and D6, state how the prime's pusher reads a takeover: for example, on a rejected prime records push it reads the takeover entry of every batten run with a local commit origin lacks (the slugs named by the `Run-Transition` trailers of the unpushed commits) and hands each taken-over run over through `pairteardown.HandOver` before any replay, and it never replays while a takeover check is open. Name `retryWeftPush`'s `runSlug != \"\"` gate in `internal/fabricengine/pushclassified.go` as the site that changes. Add a D6 acceptance criterion for a batten run taken over while the prime holds its unpushed transitions."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D7
    summary: "D7 says a paused pair's pusher pushes nothing more until resume, but the reused pre-push check skips itself when a pause covers the run, and D3's exit protocol and D5's commit and dobby restarts start a pusher on any pair ahead of origin, so the stopped run's commits are pushed on the next pass"
    fix: "In D7, state that in the pusher a pause covering the run stops the push instead of skipping the check (the `pauseCovers` branch of `skipsCheck` in `internal/loomcli/prepush.go` no longer lets a pusher's push through), and say how Finalize's skip applies in the pusher. In D3's exit protocol and seam bullet and in D5's dobby-tick bullet, add 'and no pause covers the run' to the conditions under which a pusher starts or goes on, and add a D7 acceptance criterion that a commit made after the pause is not pushed until resume."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D3
    summary: "D3 registers the pusher 'under its worktree, not under a run's slug', but a pair's worktree name is its slug and runproc keys owners by slug, so a teardown, cancel or hand-over that ends the run's processes ends the pair's pusher, against D3's own acceptance criterion"
    fix: "In D3's last bullet, name the runproc owner key the pusher registers under and show it differs from the slug: for example `runproc.HubOwner` with a pusher role that `End`, `EndSubject` and the teardown spare (as `EndSubject` already spares hub-wide detached pushes), or a distinct owner name derived from the records worktree. Keep the acceptance criterion and make it name that key."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D6
    summary: "D6 leaves to the implementer whether a hand-over from the pusher removes a loom pair: `HandOver`'s removal follows the caller's `RunPush.InsidePair`, not the worktree's kind, and loom passes true today while a pusher outside the pair would default to false"
    fix: "In D6, replace 'with the effect it has today for the worktree's kind' with the flag the pusher passes and its effect: for a loom pair's pusher `InsidePair: true`, leaving the pair for the parent's dobby as today, and for the prime's pusher false; state that the pusher then exits. Cite `internal/pairteardown/handover.go` (`RunPush.InsidePair`)."
  - id: F6
    severity: MEDIUM
    class: scope
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#Scope
    summary: "D1's 'no step waits for a push' is broken by the run's other record pushes: batten's `commitRunRecords` at driver stops and loom's `commit-records` verb push synchronously and run the pre-push check, and Scope names neither"
    fix: "Add both to Scope: either In, with D1 extended so `commitRunRecords` (internal/battencli/commitstatus.go) and the `commit-records` verb (internal/loomcli/commitrecords.go) commit and start the worktree's pusher instead of pushing, or Out, with the reason they stay synchronous."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D6
    summary: "D6 says `runfsm.CodeTipGrace` and the code-tip settle cover a records push landing before its code side 'as they do today', but today that window sits inside the waiting transition; under D1 a dobby or pre-run read in the window sees `CodeTipMissing` and the act rule answers wait while the run acts locally"
    fix: "In D6's bound, state what a read of the run during the window between the pusher's records push and its code push answers under D1, and whether the act rule's code-tip wait still applies to a run whose own pusher owes the code side; if it does not, name the change to the `CodeTipMissing` input read in `internal/pairteardown/runread.go`."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D5
    summary: "Failed-push records are keyed by repo and branch, and the prime's records branch carries every batten run, so one refused prime push stops the pusher for every batten run in the hub; D5 states the stop per worktree without that effect"
    fix: "In D5's refused-push bullet and its bound, state that a refused prime records push holds every batten run's records on this machine until the record's way forward clears it, and either accept that with a reason or narrow the stop; add the case to the D5 acceptance criterion on refused pushes."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D5
    summary: "D5 puts an unbounded owed backoff in Worktree-Teardown before the removal, and the teardown row holds the hub's prime lock across its work, so an unreachable origin during one teardown can block every other prime-lock user on the hub"
    fix: "In D5's Worktree-Teardown bullet, state whether the push to completion runs before the row takes the prime lock or inside it; if inside, state the bound on the other prime-lock users, and prefer running the push before the lock is taken (see `internal/battenshed/teardown.go`)."
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D1
    summary: "D1, the operator's decision, says no step waits for a push with no exception, while D5 makes Worktree-Teardown wait and an Auto-mode assumption calls it 'the one wait D1 allows'; the exception is stated twice and contradicts D1's own text"
    fix: "State the Worktree-Teardown exception once, in D5's bound, and add to D1 a pointer sentence that Worktree-Teardown's push before removal is the one exception, D5; drop the second Auto-mode assumption."
  - id: F11
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D4
    summary: "D4 decides what the code already does: every transition commit takes the weft write lock through `CommitRecordsPaths` with a blocking acquire, so the rationale that without it a commit could land mid-replay is a false premise"
    fix: "Turn D4 into a Constraint: 'a transition commit keeps going through `fabricengine.CommitRecordsPaths`, which takes the weft write lock that `retryWeftPush` try-acquires (internal/fabricengine/commitweftpaths.go)'; drop D4's rationale and its acceptance criterion, or keep the criterion as a regression check worded as such."
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D5
    summary: "D5's dobby restart reads 'a commit its origin tracking ref lacks', which is undefined for a records branch with no upstream yet, so a seeded Worktree-Create commit whose pusher never ran has no restart until the next transition"
    fix: "In D5's dobby-tick bullet, state that a branch with no tracking ref and at least one commit counts as ahead."
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D5
    summary: "D5's 'the pusher holds no lock while it sleeps' reads as releasing its own pusher lock too, which with D3's 'starts a pusher when the lock is free' lets a second pusher start for the same worktree"
    fix: "In D5's first bullet, replace 'holds no lock' with 'holds neither the slug's teardown lock nor the push lock while it sleeps; it keeps its pusher lock'."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D3
    summary: "D3's seam bullet and its first acceptance criterion start a pusher whenever the lock is free, while D5 says no commit starts one while a refused-push record stands"
    fix: "Add 'and no refused-push record stands for the worktree' to D3's seam bullet and to the D3 acceptance criterion 'after a commit with the worktree's pusher lock free, one pusher starts'."
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#Notes for the plan writer
    summary: "The note calls the `spawnStatusPush` variable unused, but `internal/battencli/cleanup.go` calls it in `fabricCleanupDeps.Spawn`, so a plan that drops it as dead code breaks `lym cleanup`'s push"
    fix: "Drop the word 'unused' from the first note and say that `lym cleanup`'s push spawn uses the variable."
---

### [BLOCKING] An owed push still writes the record that holds the run

**Location:** `_lym/discussion/decision-record.md` D5, D8, D3 bound
**Issue:** D3 sends the pusher through `pairteardown.PushRun` and `fabricengine.PushPairAnchored`.
That function writes a failed-push record for every owed class: `recordOwedRecordsPush` for Deferred, `teardownLockOwedError` for TeardownLock, and the transport path.
`IsPushOwed` reads exactly those classes as owed, and `ReadRun` fills `PushOwed` from the records naming the run, so the act rule answers wait.
D5's "an owed retry writes no failed-push record" and D8's "the act rule's readers keep their behaviour" therefore fail unless fabric changes, and the record names no such change.
All four lenses found this independently.

### [BLOCKING] The prime's pusher reads no batten takeover

**Location:** `_lym/discussion/decision-record.md` D3, D6
**Issue:** `retryWeftPush` runs `takeoverStop` only when `runSlug != ""`, and `PushRun` takes one slug.
A per-worktree prime pusher serves every batten run and has no slug, so a rejected prime push replays local commits onto origin's tip over another machine's takeover entry.
That breaks D6's "never replayed" and the declared `PATTERN-run-fsm` text "hands over, never replays".

### [BLOCKING] A paused pair's pusher keeps pushing

**Location:** `_lym/discussion/decision-record.md` D7
**Issue:** `skipsCheck` in `internal/loomcli/prepush.go` returns skip when a pause covers the run, and `guard` then lets the push go.
Under D1 the run commits past the reconciled point, so after the pusher pauses it, the next pass, a D3 exit re-take or a D5 dobby restart, pushes those commits unchecked to a branch origin rewound or ended.

### [MEDIUM] The pusher's runproc owner is the run's slug

**Location:** `_lym/discussion/decision-record.md` D3
**Issue:** A pair's worktree name is its slug (`seedSlug` returns the worktree name; `statusPushRequest` sets `TeardownSlug` to it), so "under its worktree, not under a run's slug" names the same owner, and `runproc.End` for the slug ends the pusher.

### [MEDIUM] D6 leaves the hand-over's pair removal open

**Location:** `_lym/discussion/decision-record.md` D6
**Issue:** `HandOver` removes the pair unless `RunPush.InsidePair`; loom passes true today. A pusher outside the pair must pick a value, and the record does not.

### [MEDIUM] Driver-stop record pushes still wait

**Location:** `_lym/discussion/decision-record.md` Scope
**Issue:** `commitRunRecords` and the loom `commit-records` verb push synchronously and are neither In nor Out of Scope.

### [MEDIUM] The code-tip window now reads as a wait

**Location:** `_lym/discussion/decision-record.md` D6
**Issue:** Between the pusher's records push and its code push, `ReadRun` reads `CodeTipMissing` while the run acts on; "as they do today" no longer describes it.

### [MEDIUM] One refused prime push stops every batten run

**Location:** `_lym/discussion/decision-record.md` D5
**Issue:** Failed-push records are keyed by repo and branch, and the prime's records branch is shared, so D5's per-worktree stop is hub-wide for batten.

### [MEDIUM] Worktree-Teardown's unbounded wait under the prime lock

**Location:** `_lym/discussion/decision-record.md` D5
**Issue:** The teardown producer holds the hub's prime lock across its calls; D5 does not say whether the owed backoff runs inside it.

### [MEDIUM] D1's no-wait rule and D5's teardown wait

**Location:** `_lym/discussion/decision-record.md` D1, D5, Auto-mode assumptions
**Issue:** The exception is stated in D5 and again as an assumption, and D1's text has none.

### [LOW] D4 restates existing behaviour as a decision

**Location:** `_lym/discussion/decision-record.md` D4
**Issue:** `CommitRecordsPaths` already takes the weft write lock with a blocking acquire; the rationale's hazard does not exist today.

### [LOW] No-upstream branches have no dobby restart

**Location:** `_lym/discussion/decision-record.md` D5
**Issue:** "A commit its origin tracking ref lacks" is undefined before the branch's first push.

### [LOW] "Holds no lock while it sleeps" covers the pusher lock

**Location:** `_lym/discussion/decision-record.md` D5
**Issue:** Read literally, it lets a second pusher start during the backoff.

### [LOW] D3's start rule omits the refused-record condition

**Location:** `_lym/discussion/decision-record.md` D3
**Issue:** D3's bullet and criterion contradict D5's refused-push stop.

### [LOW] `spawnStatusPush` is not unused

**Location:** `_lym/discussion/decision-record.md` Notes for the plan writer
**Issue:** `internal/battencli/cleanup.go` calls it.

## Focus verdicts

- Guards removed (D1, D7): the network gate moves into `PushPairAnchored`'s own admission, which holds. D7's bound covers an unreachable origin, and D2's bound covers the window before the first reconcile; the pause gap is F3.
- Tool and code facts (D4, D3, D6): D4's lock fact holds but is existing behaviour (F11); D6's rejected-push hand-over holds for a pair and fails for the prime (F2); the Scope claim that a board status write spawns a detached sync holds (`internal/boardengine/board.go`).
- Lock, exit and restart protocols: F1, F3, F4, F12, F13 and F14.
- Deferred facts: F5 (which flag `HandOver` gets) and F2 (which slug the prime's pusher reads).
- Duplication and relocation: F10; no deliberation in the Decisions needs moving to `support-log.md`.
