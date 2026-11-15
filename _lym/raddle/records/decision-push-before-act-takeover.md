## Goal

Every transition pushes its records before the run acts, and a takeover entry in the run's records hands it over without replay

Takes: runfsm's Act inputs and the batten record from run-fsm-tracked-records; fabricengine's PushPairAnchored, SpawnStatusPush, teardown-lock respawn, pushclassified's rejection path and ArchiveOwnerTips; boardengine's Sync, SpawnSync and pushOwnerCommit; statuscommit and sheddriver's records commit; landingshed's Finalize; runproc's End for a slug
Delivers: a shed transition that takes a gate admission, commits its records with the code tip it was made with and pushes records side first, code side after, before the next producer starts; an owed push retried on a backoff and shown in lym beam status, which tender's offline pause and resume intents also ride; the per-transition detached push and its teardown-lock respawn retired; synchronous Bolt writes for a run claim, a landing's done and an owner; Finalize pushing the parent branch before its transition and board done; a rejected records push that reads origin's record and hands over on another machine's takeover entry, archiving local tips and the local parent merge; the code-tip wait and its grace
Exports: the takeover entry type and its reader, tested with kit-written entries; the owed-push and code-tip facts runfsm's Act reads; fabricengine's records-first PushPairAnchored and its hand-over result

## Scope

The part changes these packages:

- internal/fabricengine (about 1000 changed lines)
- internal/battencli (about 250 changed lines)
- internal/sheddriver (about 200 changed lines)
- internal/loomcli (about 200 changed lines)
- internal/landingshed (about 200 changed lines)
- internal/boardengine (about 150 changed lines)
- internal/statuscommit (about 100 changed lines)
- internal/shedengine (about 60 changed lines)
- internal/beamcli (about 60 changed lines)
- internal/fabriccli (about 40 changed lines)
- internal/tendercli (about 40 changed lines)
- internal/tenderengine (about 40 changed lines)
- internal/pauserecord (about 40 changed lines)
- pattern (about 30 changed lines)

## Decisions

### D2 — A transition pushes its records before the run acts

- A shed transition commits its records as today, the files D3 moves into tracked records included, then pushes them synchronously:
  both sides in a task pair, records only in the prime.
  The records commit names the code tip the transition was made with (D4).
  The next producer starts only once every side has landed.
- The pair push itself, `fabricengine.PushPairAnchored`, sends the records side first.
  It sends the code side only once the records side has landed, and none when the records side fails or is rejected,
  so the order holds for every caller of a pair push, a transition's and `lym loom commit-records`' among them.
  Today that push sends the code side first.
  A code side withheld this way is reported with the records side's failure, and the caller's next pair push sends it.
- A Finalize's parent-side merge pushes the parent branch before its transition's records and before the landing's board `done` (D4).
- Every detached status push (`fabricengine.SpawnStatusPush`) that carries a run transition retires:
  the per-transition spawn behind `internal/statuscommit/statuscommit.go`,
  and the respawn of a pair's push that exited on a held teardown lock (`internal/fabricengine/teardownlock.go`).
  A transition's push that meets a held teardown lock waits for its release and pushes before the next producer starts.
  A wait that outlasts the push lock wait (`fabricengine.StatusPushLockWait`) turns the push owed;
  that releases the admission below, so a pause written meanwhile reaches the process again.
  Fabric's pause deferral of a push no longer applies to a transition's push, which is synchronous;
  it stays for the detached pushes below.
  A transition takes a network gate admission (`pauserecord.Admit`) before its commit,
  and releases it once every side has landed or its push turns owed,
  so a pause or hold written after the commit stops the run's next producer and never that push's first attempt.
  An owed push's retries wait at the gate like any network call, so a pause holds them, and the run waits on the owed push meanwhile.
  `lym loom commit-records` keeps its synchronous push.
- The anchor change's push (`internal/fabricengine/anchorchange.go`) and tender's prime push (`internal/tendercli/deps.go`) stay detached.
  Neither carries a run transition: the first pushes a pair's anchor move,
  and the second pushes only commits no run waits on, since every transition pushed before its run acted.
- A push that fails for a transient reason, such as an unreachable origin, leaves the run waiting at that transition with the push owed.
  It retries on a backoff under `PATTERN-daemon-wakeups` and starts no producer meanwhile.
  `lym beam status` shows the owed push and its last error, and the existing failed-push record and creel notice stay.
- A rejected push reads origin's tip for a takeover first (D4), and replays the records as today only when it finds none.
- A Bolt write that carries a decision pushes synchronously before its writer goes on:
  a run claim, the board `done` of a landing, an owner, a hold (D7) and the registry (D5).
  A board write that only displays a run status stays on the detached board sync.

Bound: no run acts while origin is unreachable.
Work an agent made inside a step and has not reached its transition yet is on this machine only;
a stop there loses nothing pushed, and the step runs again from its start.

Rationale: the operator requires the whole state on GitHub at all times, and today a push can lag or fail with nothing waiting for it (state map 1.3, 7).
A push costs two network round trips or more (`internal/fabricengine/pushclassified.go`), small beside an agent step.

### D4 — A takeover entry in the run's own records hands the run over

- Adopt writes a takeover entry, naming the new owner machine, into each moved run's batten record and child record, each on its own branch, and pushes it (D8).
- When a run's records push is rejected, a transition's or any other caller's of the pair push (D2),
  lym fetches the branch and reads the run's own record from origin's fetched tip, before any replay.
  When that record holds a takeover entry naming another machine that this machine has not yet read, the run hands over without replaying:
  this machine pushes no code side for the rejected push, ends its processes for the slug (D9),
  pushes the code tip the run's last transition on origin names when origin's code branch lacks it, since that transition stands,
  archives the local tips as the old owner's archive does (`fabricengine.ArchiveOwnerTips`), with uncommitted files first committed onto the archived tip,
  archives its local parent branch's tip the same way when this run's parent-side merge alone puts that branch ahead of its upstream,
  and resets that branch to its upstream,
  pushes the archive tags, and posts one notice naming the machine and any pause or cancel intent it dropped (D3).
  It pushes nothing more for the run, so its code commits after the takeover reach origin only under the archive tags.
  A run `landing` past its parent-side merge hands over too, and its new owner resumes the Finalize from origin's records.
  The Finalize reads the owner from a fetched Bolt before its parent-side merge and before each later push,
  and hands over when another machine owns the run, so the old owner writes nothing to the parent's branch after it reads the takeover.
  After the merge into the local parent branch, the Finalize pushes the parent branch, then commits and pushes its transition,
  then writes the board `done` (D2); today it marks the board `done` before the parent push (`internal/landingshed/finalize.go`).
  Under the push skip, which pushes nothing, the board `done` follows the local merge.
  So whenever origin's records or board read the run past its merge, origin's parent branch holds the task's work,
  and the adopter reads that from its fetch (D1).
  A takeover between the local merge and the parent push leaves origin's parent branch without the work:
  the adopter reads the run as before its merge and merges again,
  and the old owner's hand-over archives and resets its local parent branch as above,
  so a later landing on that machine finds the branch level with its upstream.
  A parent push rejected over an origin parent branch that already holds the task's work archives the local merge,
  resets the local parent branch to origin's, and goes on past the merge.
- Each transition's records commit names the code tip it was made with (D2).
  After a takeover, origin's code branch therefore moves only to a tip a pushed transition names,
  so the adopter's later code push meets no other machine's commit.
  Until origin's code branch holds the tip the run's last transition names, the act rule answers `wait` (D1),
  and `lym beam status` names the tip and the machine that owes it;
  that machine's next hand-over or catch-up (D6) pushes it,
  and the owner's dobby fast-forwards its local code branch to it before the run's next pass.
  A tip still missing after the code-tip grace is given up:
  the owner commits and pushes a records commit that returns the run to its previous transition,
  and the step runs again from there, as D2's bound says a stopped step does.
  A machine owing a tip pushes it only while the run's last transition on origin names it, so a given-up tip never lands.
- A rejection whose fetched record holds no such entry, such as one over commits of other runs on a shared records branch like `main-weft`,
  replays and pushes as today.
- An adopter's own takeover push that is rejected fetches, then re-reads its hold and the subjects' owner from a Bolt it fetches.
  When its hold is gone or either names another machine, it stops, naming the subjects' owner it reads from that Bolt (D8).
  Otherwise it writes the takeover entry again onto origin's tip and pushes again;
  it never replays its own commit over a record the old owner changed, and it never hands over.

Bound: transitions the old owner pushed before the takeover landed stand, and the adopter goes on from them.
The old owner's step in flight runs to its next transition, which the takeover rejects;
its result is archived under a tag, not lost and on neither branch.

Rationale: a takeover entry in the run's own record is a compare-and-swap per run that holds on records branches many runs share,
and needs no extra fetch per transition.

### Added after Discussion (arbiter, 2026-11-15): No run acts while origin is unreachable

Decision: D2's bound stands as written: no run acts while origin is unreachable, and an owed push holds the run at its transition until it lands. This answers review finding F15 (no-offline-progress-unconfirmed). The writer removes the Open risks bullet 'D2 stops every run while origin is unreachable. The parent confirms the operator accepts no offline progress.' and changes nothing else in D2, D3 or D6 for it.

Rationale: The board entry's operator requirement (2026-11-15) says the whole state of lym and of every run is stored on GitHub at all times, and its Direction says each transition writes and pushes its record before it acts. A run acting offline would leave a state on this machine only, which that requirement rules out; the support log's rejected alternative 'letting a run act offline and pushing later' already says so.

### Added after Discussion (arbiter, 2026-11-15): No lym adopt --release; a hold clears only at a green check

Decision: Answers review finding F13 (release-verb-against-operator-decision): no verb lets the hub go on past an adopt hold before an adopt's check is green. The writer drops 'lym adopt --release' from D7 (the upgrade bullet and the Bound), D8 (the release paragraph, the stop-on-release lines, the Bound), the Constraints (the shell-guard row and the refusal-way-forward line), the declared PATTERN-dobby-runs change and the acceptance criteria that name it, and removes the Open risks bullet on --release. D8 instead states how a hold whose adopter stopped is cleared without the hub going on: an operator's lym adopt of the same subjects on another machine, given an explicit flag the writer names, takes over a hold another machine wrote by rewriting it to name this machine, then runs its own steps and check, so the hold clears only at that adopt's green check or, as an adopt-fix hold, at the fix task's green re-check. That flag is denied to agents by a shell-guard row and no text an agent reads names it; the stopped adopter, if it returns, stops at its next hold re-read as D8 already says. At the fix-try limit the hold stays until the failing tests are fixed and a re-run of lym adopt checks green.

Rationale: The board entry's Operator decisions (2026-11-15) say nothing else happens in the hub, no run start, no dobby pass and no orch action, until adopt has finished and every failing test it found is fixed. A release that lets the hub go on with failing tests, or without the check of an adopter that stopped, contradicts that directly, as the record's own Open risks line says. A hold taken over by a second adopt keeps the stopped-adopter case recoverable while the hub still waits for a green check.

### Added after Discussion (parent, 2026-11-15): No orch transcript mirror (F12)

Decision: Drop D10's transcript mirror to the weft repository and every cross-machine restore of the orch's transcript. The orch never needs to continue its conversation on another machine: an orch started on a machine that has no local transcript for the worktree launches fresh with the carry-over. Keep only the same-machine fix: an orch start on the machine that holds the transcript resumes that session and never falls back to a fresh launch because an older live process holds it (A6, A7).

Rationale: Operator, 2026-11-15: the orch shall not need to continue anything on a new machine. Also settles F12: no orch transcript goes to the public weft repository.

### Added after Discussion (parent, 2026-11-15): Orch up during a hold (F14)

Decision: The orch is up while an adopt or adopt-fix hold stands, as D8 has it: every verb the act rule guards refuses except on the fix task and on a landing past its parent-side merge, and the orch handles the fix task's halts. The board's 'no orch action until adopt is done' means no orch action on held runs.

Rationale: Operator, 2026-11-15, answering F14: yes, the orch may be up during a hold with those refusals.

### Added after Discussion (arbiter, 2026-11-15): Round 3 review findings answered by the round 3 fixes

Decision: The Discussion review ends at its hard cap with round 3's findings answered by the record as the round 3 fixer left it, and no further edit is owed for them. F1: D1 reads past the parent-side merge from origin's fetched parent branch, D2 and D4 push the parent branch before the Finalize's transition and its board done, and D4 covers a takeover between the local merge and the parent push. F2: --release is gone; D8's lym adopt --take-hold, D7's hold subjects, the shell-guard row and their acceptance lines replace it. F3: D10 keeps only the same-machine resume. F4: Open risks reads None. F5: D2 puts records-first order in fabricengine.PushPairAnchored for every caller, and D4 hands over on any caller's rejected records push. F6: D3 deletes a cancelled run's batten record in the same step as every pair removal, and D1 reads a leftover cancelled record whose task branch is gone from origin as no record. F7: D2 turns a teardown-lock wait past StatusPushLockWait into an owed push, and D7 writes the upgrade hold only when a subject remains to re-run.

Rationale: Each fix in the record is the one the round 3 review proposed, and the code it names holds: finalize.go marks the board done (line 445) before pushParent (line 786), replay.go's finalizeEntered and ParentHoldsTaskWork read local state, PushPairAnchored iterates a code and a records side, battencli removePairAndBranch discards the task branch on origin, pauserecord.Admit, fabricengine.StatusPushLockWait and pairteardown Request.Local exist. The fixer disputed and deferred nothing.

## Constraints

- `PATTERN-owner-machine`: the owner stays on roots and beams, written only by `boardengine`'s setter, read through `hubgeom.ParentOf`.
  The rest of a Finalize past its parent-side merge, as origin's parent branch shows it,
  finishes on the run's owner, the adopter after a takeover (D1, D4).
- `PATTERN-pair-remote`: the code branch still goes along with a pair's records push, now once the records side lands (D2),
  and a refused push still leaves a failed-push record and a notice.
  A catch-up's local pair removal (D6) goes through `internal/pairteardown`'s local mode and deletes no origin branch.
- `PATTERN-dobby-runs`: runs are still started only by the parent's dobby, which starts only with its orch bar `--no-orch`.
- `PATTERN-lym-files`: every new tracked file sits under `_lym` and every cache under `.lym`, both named in `internal/lymdirs`.
- `PATTERN-tolerant-records`: the new tracked records and the Bolt `machines` and `holds` keys read tolerantly and are written through `internal/state` or the board store.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: the synchronous push, the takeover entry's push and the archive run in `internal/fabricengine` through `gitexec` and `gitrepo`.
- `PATTERN-shuttle-provider-seam`: Claude's transcript paths and the resume check stay in `internal/shuttleengine/claudeengine`.
- `PATTERN-child-processes`: the orch's Claude process registers in `internal/runproc` (D9), and D9's ends go through it.
- `PATTERN-daemon-wakeups`: D2's and D6's retry waits wake at most once a second and back off while idle.
- `PATTERN-refusal-way-forward`: every refusal the act rule, the holds, D3's owner refusal, D5's clash refusal and D9 add
  names its way forward and a test reaches it;
  none an agent reads names `lym adopt --take-hold`, which a shell-guard row denies agents (D8).
- `PATTERN-shell-guard-table`: the deny of `lym adopt --take-hold` is a row of `internal/shellguard`'s table.
- `PATTERN-code-conventions`: the orphan grace and the code-tip grace are each one named constant.

## Auto-mode assumptions

- The code-tip grace is 30 minutes (D4) and the orphan grace 24 hours (D11).

## Open risks

## Acceptance criteria

- D2: a shed transition in a task pair pushes its records side, then its code side once the records side landed, and starts the next producer only after both landed on origin.
- D2: a pause written between a transition's commit and its push leaves that transition on origin, and the run's next producer does not start.
- D2: a transition's push that waits on a held teardown lock past the push lock wait turns owed, and a pause written then holds the process's network calls.
- D2: with origin unreachable, a transition leaves the run waiting with the push owed, starts no producer, and goes on once origin answers.
- D3: with origin unreachable, `lym tender pause <slug>` holds the run's processes here at once, and the pause intent is on origin once origin answers.
- D4: a records push rejected over a takeover entry naming another machine ends this machine's processes for the slug, pushes an `archive-owner/` tag holding its local commits and uncommitted files, and pushes nothing to the branch.
- D4: a push rejected over another run's commits on a shared records branch replays and pushes, and the run goes on.
- D4: a records push rejected over a takeover entry runs no replay, so no replay conflict stops the hand-over.
- D4: after a takeover, a landing past its parent-side merge is driven on exactly one machine, the adopter, which finishes its Finalize.
- D4: after a takeover, the old owner's next transition leaves origin's code branch unchanged, and its code commits are under its archive tag.
- D4: a run whose last transition names a code tip origin's code branch lacks waits until that tip lands, then goes on from it; one still missing after the code-tip grace returns to its previous transition and runs the step again, and the owing machine's later push of that tip leaves origin's code branch unchanged.
- D4: an old owner past its parent-side merge that reads the takeover pushes nothing more to the parent's branch, and the adopter finishes the Finalize.
- D4: an old owner that merged into its local parent branch and was taken over before pushing it leaves origin's parent branch without the work; the adopter merges and lands the run, origin's parent branch holds the work once, and the old owner's local parent branch is level with its upstream, its merge under an archive tag.
- D4: a Finalize's board `done` reaches origin only after its parent branch push landed.
- D4: after a takeover, the old owner's `lym loom commit-records` hands over and leaves origin's code branch unchanged.

## PATTERN changes

- `PATTERN-run-fsm` — adds: Reading or acting on a run's lifecycle state: only `internal/runfsm` (new)'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along after the records side, both before the run acts; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.
