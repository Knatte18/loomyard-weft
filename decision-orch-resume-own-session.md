## Goal

The orch ends its own earlier Claude process and resumes its session, never launching fresh over a live holder

Takes: internal/orchcli/start.go's launchOrch with its fresh-launch fallback, runproc's roles and End functions, claudeengine's CheckResume, the orch watcher's owned-elsewhere read and shuttleengine's carry-over; PATTERN-test-only-code holds, so every new function has a production caller
Delivers: the orch's Claude process registered in runproc under an orch role keyed by the worktree; an orch start that ends its own registered holder and resumes the same session id, refuses an unregistered holder naming its pid, and launches fresh with the carry-over only when no local transcript exists, saying so; the orch watcher ending every process of a subject owned elsewhere, the dobby last
Exports: runproc's orch role and its registration call, and one function that ends every process this machine runs for a subject, which the orch watcher calls here and the dobby's handover and the catch-up call in bolt-holds-catch-up

## Scope

The part changes these packages:

- internal/orchcli (about 350 changed lines)
- internal/runproc (about 120 changed lines)
- internal/orchengine (about 50 changed lines)
- internal/shuttleengine (about 30 changed lines)

## Decisions

### D9 — A moved subject's processes end, and the orch resumes its own session

- When this machine reads a subject owned elsewhere, through the dobby's handover, the orch watcher, a catch-up or D4,
  it ends every process it runs for the subject: run strands, step loops, the orch strand with its Claude process, and the dobby last.
  A run `landing` past its parent-side merge ends here too, and its owner finishes it (D4).
- The orch's Claude process is registered in `internal/runproc` at launch, under an orch role keyed by the worktree;
  the registration outlives the strand's removal and is pruned when the process dies.
  Ending the subject's processes ends it through `internal/runproc`.
- At an orch start whose resume `claudeengine.Claude.CheckResume` refuses for a live holder,
  lym ends the holder when `internal/runproc` registers it as this worktree's orch, then resumes the same session id.
- A holder lym has not registered is refused, naming its pid and the way forward: exit that session, then `lym orch start`.
  The orch does not launch fresh over it.
- An orch Claude process the pre-upgrade lym launched is unregistered too,
  so the first orch start after the upgrade refuses over it, naming its pid, and the operator exits that session once.
- An orch start on a machine with no local transcript of its session launches fresh, with the carry-over of `internal/shuttleengine/carryover.go`,
  and the start says so.

Rationale: on both recorded adopts the orch's own earlier session, still alive on the adopting machine after its strand was removed, held the transcript,
and `internal/orchcli/start.go` fell back to a fresh launch on any resume error (A6, A7).

### D10 — The orch's transcript stays on its machine

- The orch's session id stays in the machine-local orch state (`orchengine.State`), and no tracked record names it,
  since no other machine resumes it.
- Nothing carries the orch's transcript between machines; an orch start without a local transcript is D9's fresh launch.

Rationale: the operator decided the orch never continues its conversation on another machine,
so no transcript leaves the machine that holds it.

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

- D9: an orch start whose session is held by this worktree's own earlier orch process, its strand already removed, ends that process and resumes the same session id.
- D9: an orch start whose session is held by a process lym has not registered refuses, naming the pid, and launches nothing fresh.
- D10: an orch start on a machine with no local transcript of its session launches fresh with the carry-over, says so, and writes no tracked record of the session id.
