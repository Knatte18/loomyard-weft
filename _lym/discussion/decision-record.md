## Goal

One run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked

Takes: shedengine.Status as the batten and child run record, boardengine's run status helpers, battencli's cancel marker, tenderengine's pause and resume, pullrequest's record and landingshed's fixer attempt counts, all as they stand; PATTERN-test-only-code holds, so every new function has a production caller
Delivers: internal/runfsm deriving every run state of D1 from tracked records alone and answering the act rule from its inputs, with the owner, hold, owed-push and code-tip inputs passed in by callers; the dobby's pass, lym batten run, lym loom resume and lym tender resume asking it; the cancel intent with its finish and the run pause intent in the batten record, committed and pushed with today's pushes; a cancelled run's record deleted with its pair; the PR record and fixer counts in the pair's records; the origin-owed marker stays until bolt-holds-catch-up
Exports: runfsm's State type and order, its Derive over tracked records and its Act rule with an inputs struct that later parts fill; the batten record's cancel and pause intent fields and their writers under shedrun's tracked paths

## Scope

The part changes these packages:

- internal/runfsm (about 900 changed lines)
- internal/battencli (about 450 changed lines)
- internal/dobbyengine (about 230 changed lines)
- internal/landingshed (about 250 changed lines)
- internal/loomcli (about 150 changed lines)
- internal/tenderengine (about 150 changed lines)
- internal/tendercli (about 60 changed lines)
- internal/shedrun (about 80 changed lines)
- internal/pullrequest (about 60 changed lines)
- internal/shedengine (about 40 changed lines)
- internal/boardengine (about 40 changed lines)
- internal/pairteardown (about 40 changed lines)
- internal/lymdirs (about 10 changed lines)
- pattern (about 10 changed lines)

## Decisions

### D1 — One run FSM, derived from tracked records by one package

A new leaf package `internal/runfsm` (new) declares a run's lifecycle states, their order, and the act rule.

A run's state is derived from tracked records alone: the board entry, the Bolt holds and owners (D5, D7), the batten run record, and the loom child run record.
A record is read from the local worktree, or from origin's branch when this machine has no pair for the run.
The states, highest precedence first, each taken when its condition holds and no earlier one does:

1. `landed`: the batten record `done`, or no batten record and the board status `done`, as the dobby's done-run clear leaves a landed run.
2. `cancelled`: a cancel intent with its finish (D3), the finish saying whether the pair was kept.
3. `cancelling`: a cancel intent (D3) without a finish.
4. `paused`: a run-scope pause intent (D3), or a shed state `paused`.
5. `halted`: the child run `blocked`, `failed` or `awaiting`.
6. `landing`: the child run at `Finalize` or a later row.
7. `running`: the batten record at `Run-Shed` or later.
8. `starting`: a batten record at a row before `Run-Shed`.
9. `queued`: a released task with an empty board status and no batten record.

A `cancelled` batten record whose task branch is gone from origin, which a crash between a pair removal and the record's deletion leaves (D3),
counts as no batten record, so a released task with one reads `queued`;
a kept pair's task branch stays on origin, so its record reads `cancelled` on every machine.
A `landing` run is past its parent-side merge when origin's parent branch, as this machine last fetched it, holds the task branch's work;
the local parent branch never decides it, since a merge there reaches origin only with the parent push (D4).
Under the push skip, which pushes nothing to origin, the local parent branch decides it (D4).

A pair stands or not beside each state; a run in the states D11 names whose pair stands owes a teardown (D11).

The act rule answers, for one run on this machine, `act`, `wait` or `hand_over`.
Its inputs are the run's state, the owner read through `hubgeom.ParentOf` and the registry (D5), the Bolt holds (D7),
whether origin's code branch holds the code tip the run's last transition names (D4),
whether origin's fetched parent branch puts a `landing` run past its parent-side merge (below),
and two machine-local facts: a push owed with origin unreachable (D2), and this machine's pause record coverage.

- `wait` when a push is owed and origin is unreachable.
- `hand_over` when another machine owns the run; its owner finishes even a `landing` past its parent-side merge (D4).
- `wait` when origin's code branch lacks the code tip the run's last transition names (D4).
- `act` when the run is `landing` past its parent-side merge and no pause covers it, so it finishes under a hold.
- `wait` when a hold covers it, a pause covers it, the state is `paused` or `cancelling`, or the state is terminal.
- `act` otherwise.

Every verb or loop that acts on a run asks the act rule first, and every reader of a run's state takes it from `internal/runfsm` (new).
`lym tender resume <slug>` and `lym batten cancel` set the run's own pause and cancel intent aside when they ask,
and `lym batten cancel` sets a hold aside too, so the operator can resume or cancel a held run.
The board's `<state> · <row>` status is a display written from the FSM state; no decision reads it but `queued`'s empty status and `landed`'s `done` with no batten record.

Rationale: the state map (section 6.4) finds six vocabularies that disagree on `paused`, `running`, `done` and the halted states.
One package over tracked records gives every machine the same state from the same pushed bytes.

### D3 — Run decisions move into tracked records; the rest is cache

These move from a pair's or the hub's `.lym` into tracked records under the run's `_lym/shed/<slug>/`:

- The cancel intent, in the batten record: when it started, the machine, and remove or keep.
  It replaces the cancel marker of `internal/battencli/cancelmarker.go`; the cancel's lock stays local.
  `lym batten cancel` commits and pushes it before it ends anything, or acts offline as below.
  A cancel keeps the batten record, in place of today's removal of the run's records,
  and writes its finish into it, the time it ended and remove or keep, committed and pushed when the cancel ends.
  The finish sets the shed state `cancelled`, never `done`,
  so the dobby's done-run clear (`battencli.ClearDoneRun`), which deletes the task branch on origin, never takes a cancelled run.
  Every path that removes a cancelled run's pair deletes the slug's batten record in the same step as the pair removal:
  the cancel's own removal under `--remove`, D11's dobby teardown and `lym fabric remove <slug>`;
  a slug released again then reads `queued`.
  `lym batten cleanup` only repairs a record a crash left: it removes a `cancelled` record whose pair is gone,
  and refuses one whose pair stands, naming `lym fabric remove <slug>` as the way forward.
  A landed run's record goes at the dobby's done-run clear as today, and the run reads `landed` from its board `done` (D1).
- A run-scope pause intent, in the batten record: who paused and when.
  `lym tender pause <slug>` writes it and `lym tender resume <slug>` clears it, each committing and pushing before it returns.
  Each also writes this machine's pause record entry as today, which the network gate and the turn interruption read.
- The pull-request record of `internal/pullrequest/record.go`, whole, in the pair's records; a write of it commits and pushes at once.
- The fixer attempt counts of `internal/landingshed/fixerattempts.go`, in the pair's records, committed with the transition.

A pause or cancel of a run another machine owns is refused, naming that machine, so only the owner writes a run's records.
Its way forward is the same verb on that machine, or `lym adopt` of the run's subject here first.
The verb reads the owner from a Bolt it fetches first.
A machine that becomes a run's owner writes its pause record entry from the run's pause intent at its catch-up (D6).

With origin unreachable, each of these verbs acts here at once and owes its push under D2's retry:

- `lym tender pause <slug>` writes this machine's pause record entry, so the run's processes stop, and commits the intent.
- `lym tender resume <slug>` commits the cleared intent; the run waits on the owed push (D1) until it lands.
- `lym batten cancel` ends the run's processes and strands and commits the intent.
  Its output names as way forward a re-run of `lym batten cancel <slug>` once origin answers,
  which pushes the intent and finishes the cancel, its finish and any pair removal.

Offline, the owner refusal reads the local Bolt, and a takeover the owed push meets is D4's hand-over.
Bound: until the owed push lands, the intent is on this machine only, and another machine reads the run unpaused or not cancelled.
When the owed push meets a takeover, the hand-over drops the intent, and its notice names each pause or cancel intent it dropped,
to repeat on the owner machine.

The origin-owed marker of `internal/battencli/resumereconcile.go` retires: under D2 and D6 no pass acts without origin.

Everything else run-related under `.lym` is a cache that a fresh machine starts without or rebuilds:
the merge record, the driver park marker and resume report, the batten per-row marks, the dobby's `beam/state.json`,
shuttle run directories, reed state, loom scratch, gate and verify records, the parent lock, the run process registry,
the creel journal, adopt check views, failed-push records, the hibernate request, and the pause record.
An absent cache reads as empty and refuses nothing.
A sent-notice memory lost this way can repeat a notice or a revive once on the new machine.

Tracked values that hold only on the machine that wrote them, the Claude session ids in webster's state and the pull-request record,
and absolute paths in step envelopes, are read as such:
a session id whose transcript is absent here starts that session fresh, and an absolute path is display only.

Rationale: the state map (section 7) lists these as the run-relevant state held only under `.lym`,
and `PATTERN-lym-files` already says a run depends on nothing that lives only there.

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

### Added after Discussion (parent, 2026-11-15): A shed pause is an aside of the act rule

Decision: Besides the pause intent, the cancel intent and the hold, a run whose shed state is paused is an aside of D1's act rule: lym batten run, lym batten step, lym loom start, lym loom resume, lym batten cancel and lym tender resume may act on it.

Rationale: Without the aside, the act rule waits on a shed state of paused, so a shed pause's own resume verb would refuse itself and the run could never leave the pause.

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

- D1: for each state, a fixture of tracked records alone reads that state through `internal/runfsm` (new), with every `.lym` cache absent; a fixture meeting two states' conditions reads the one earlier in the order.
- D1: a slug with no batten record and a board status `done` reads `landed`.
- D1: the act rule answers `hand_over` for a run another machine owns, a landing past its parent-side merge included; `act` for one `landing` past its parent-side merge under a hold unless a push is owed offline or a pause covers it; `wait` under a hold, a pause, the state `paused` or `cancelling`, an owed push with origin unreachable, or a last transition whose code tip origin's code branch lacks; and `act` otherwise.
- D1: a `landing` run whose local parent branch holds the task's work while origin's fetched parent branch does not reads as before its parent-side merge.
- D1: a released task whose `cancelled` batten record outlived its removed pair, its task branch gone from origin, reads `queued` on every machine; a kept cancel's record reads `cancelled` on a machine without its pair.
- D1: the dobby's pass, a hand `lym batten run`, `lym loom resume` and `lym tender resume` each refuse a run the act rule answers `wait` or `hand_over` for, naming the way forward.
- D3: after `lym batten cancel` returns, the cancel intent is on origin in the run's batten record; a cancel interrupted on one machine reads `cancelling` on another.
- D3: `lym tender pause <slug>` puts the pause intent on origin and holds the run's processes here; a pause of a run another machine owns is refused naming that machine and, as way forward, the same verb there or `lym adopt` here first.
- D3: a finished cancel with keep leaves the batten record with its finish on origin, and the run reads `cancelled` with keep.
- D3: `lym batten cancel <slug> --remove` deletes the slug's batten record with its pair; the task released again is started by its dobby.
- D3: a finished cancel's batten record has the shed state `cancelled`; the dobby's done-run clear leaves it and its task branch, and `lym batten cleanup` removes it once its pair is gone and refuses it, naming `lym fabric remove <slug>`, while its pair stands.
- D3: a pull-request record and the fixer attempt counts written on one machine are read on another after a pull.
- D3: a run whose `.lym` is removed reads the same FSM state as before the removal.

## PATTERN changes

- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`.
