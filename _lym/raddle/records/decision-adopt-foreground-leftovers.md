## Goal

lym adopt runs in the foreground over pushed state, with --take-hold, and the dobby tears down or reports leftover pairs

Takes: the holds, registry, catch-up and owner setter from bolt-holds-catch-up; the takeover entry type, its reader and the synchronous pushes from push-before-act-takeover; runfsm's states; adoptcli's fix loop, check, deploy, archives and here paths; the dobby's ended-run teardown and pairteardown
Delivers: lym adopt running hold, owner, pull, takeover, catch-up, deploy, workspace, check and green clear in the caller's process, re-runnable from tracked state, with the takeover entry writer and its rejected-push retry, and --take-hold denied to agents by a shell-guard row; the worker, adopt record and its readers, verdicts, pause step, the adopt and adopt-hold pause callers, --keep-local, --drop-local and --discard retired, with the upgrade's first read turning standing adopt entries into pause intents and one Bolt hold; the dobby tearing down landed and unkept cancelled pairs, listing kept pairs, writing kept-cancel records for pre-upgrade kept pairs, and tearing down idle orphans after the orphan grace; lym tender resume pushing no pair; the hub workspace file regenerated after each teardown
Exports: adopt's foreground step runner, the takeover entry writer and the orphan grace constant

## Scope

The part changes these packages:

- internal/adoptcli (about 2650 changed lines)
- internal/adoptrecord (about 400 changed lines)
- internal/dobbyengine (about 350 changed lines)
- internal/fabricengine (about 150 changed lines)
- internal/beamcli (about 150 changed lines)
- internal/orchcli (about 80 changed lines)
- internal/pauserecord (about 80 changed lines)
- internal/battencli (about 60 changed lines)
- internal/shellguard (about 30 changed lines)
- internal/tendercli (about 30 changed lines)
- internal/pairteardown (about 30 changed lines)
- cmd/lym (about 10 changed lines)
- pattern (about 10 changed lines)

## Decisions

### D8 — `lym adopt` runs in the foreground: hold, owner, pull, takeover, check

`lym adopt <subject>...` runs every step in the caller's process and prints each step as it ends.
It starts no detached worker and writes no adopt record.
Each step reads tracked state, so a re-run after an interruption goes on from where the state stands.

1. One Bolt commit writes an `adopt` hold, this machine as the subjects' owner, and this machine's registry entry, and pushes it.
   A hold another machine wrote refuses the adopt, unless it runs with `--take-hold` (below), naming the writer machine and the hold's time,
   and as way forward waiting for that machine's adopt to clear it, or asking the operator when that machine has stopped.
   A rejected push reads Bolt again; an owner of these subjects, or any hold, that another machine wrote since the read
   refuses the adopt naming that machine; otherwise it replays and pushes.
   A registration D5 refuses refuses the adopt with D5's way forward.
2. For each moved run, it fetches the run's branches, archives this machine's local tips that origin lacks as D4 does, resets them to origin's,
   and adds the pair from origin when this machine has none.
3. It writes the takeover entry (D4) on each moved run's records branches and pushes it.
   Once it lands, it fast-forwards each moved run's local records branches to origin's tips, which the takeover entry now heads,
   and its local code branch to origin's code tip.
   A code tip the last transition names that origin still lacks is the owner's dobby's to fast-forward to once it lands (D4).
4. The catch-up (D6) over the whole hub.
5. The deploy, as today.
6. The cache it rebuilds: the hub workspace file.
7. The check: the hub's `adopt` gate runs in each subject's worktree, in the foreground, with its output on the terminal.
8. A green check clears the hold and runs `orchcli.BeamUp` for each subject; the runs go on by the act rule, with no hand lift.
   A failing check files a fix task through the existing fix loop (`internal/adoptcli/fixloop.go`, `fronttask.File`),
   replaces the hold with an `adopt-fix` hold naming it, runs `orchcli.BeamUp`, and exits non-zero naming the task.
   The task's landing runs the check again; green clears the hold, and failing files the next task within `adopt_fix_tries`.
   Each subject's try count lives in the `adopt-fix` hold, rewritten with each filing,
   so a re-check on any machine reads the same count against that subject's limit.
   An adopt that took over an `adopt-fix` hold keeps its standing fix task instead (below).
   At the limit the hold stays and the dobby's notice names the failing tests and the fix task;
   the hold clears only once the failing tests are fixed and a re-run of `lym adopt` checks green.

`lym adopt --take-hold <subject>...` (new) takes over a hold another machine wrote, such as one whose adopter stopped.
Its step 1 refuses a hold another machine wrote unless that hold names exactly these subjects, naming the hold's subjects as the way forward.
With no standing hold, or one this machine wrote, it runs as `lym adopt` does.
Otherwise its Bolt commit rewrites the hold to name this machine, keeping its kind, fix task and try counts,
writes this machine as the subjects' owner, and pushes; the adopt then runs steps 2 to 8.
Under an `adopt` hold its green check clears the hold.
Under an `adopt-fix` hold the fix task stands, a failing check files no second task,
and the hold clears at the adopt's green check or the fix task's green re-check.
An adopt re-reads its own hold from Bolt before each step from step 2 on,
and stops once the hold is gone or names another machine, naming the subjects' owner it reads from that Bolt and starting nothing,
so a taken hold leaves one adopter, and the stopped adopter's subjects go on under the adopt that took them.
A shell-guard row denies `--take-hold` to every agent, and no refusal or notice an agent reads names it;
adopt's help names it.

Kept: `--here` (behind `lym s`, now in the foreground), `--archives`, `--no-orch` and `--all`.
Retired: `--keep-local`, `--drop-local`, `--discard`, `lym adopt worker`, the adopt record, the held-back project verdicts, and the adopt pause step.

Bound: adopt works from pushed state alone and never asks about another machine's local state;
the old owner's unpushed work reaches the adopter only through D4's archive tags, which `--archives` lists,
besides the code tip of a transition already on origin (D4).
While a hold stands the orch is up, but every verb the act rule guards refuses except on the fix task and a landing past its parent-side merge.
The hold's pause entries cover only held runs (D7),
so the orch, adopt's deploy and check with their children, and the operator's `lym batten cancel` go on.
`--take-hold` skips no check and lets no run go on before a green check: it moves the hold and its adopt to this machine.
Only the operator's own shell runs it.

Rationale: the operator decided adopt runs in the foreground and that nothing happens in the hub until it is done and every failing test it found is fixed.
D2 to D4 leave no local state for adopt to reconcile, which removes the stops A2, A3, A5, A12 and A13.

### D11 — Leftover pairs are torn down or reported

- The dobby tears down, through `internal/pairteardown`, the pair of each run in the FSM state `landed`, or `cancelled` without keep,
  with no pause or hold covering it and no other machine owning it.
  A cancelled run's batten record goes in the same step (D3).
  This widens the ended-run teardown of `internal/dobbyengine/ended.go`,
  whose conditions read these in place of the cancel marker and the pause record.
- A cancelled run whose pair was kept stays: `lym beam status` lists it with its age,
  and each dobby start posts one notice listing kept pairs and naming `lym fabric remove <slug>`.
- A pair kept by a cancel before the upgrade has no batten record left.
  The first dobby start after the upgrade writes, for each pair whose slug has a board entry, no batten record and no live process,
  a batten record holding a finished cancel with keep, so the pair is listed as kept and never read as `queued` or as an orphan.
  The board entry must be unclaimed, as a cancel leaves it, so a pair whose claim stands is never read as kept.
- A pair with neither a board entry nor a batten record, such as a bench's or a trial's, is an orphan.
  The dobby posts a notice at first sight saying when it goes, and tears it down no earlier than the orphan grace after that notice,
  and only when neither branch has had a commit for the orphan grace and no process of its slug lives.
  Uncommitted files are committed onto the archived tip first.
- `lym tender resume` pushes no pair, since D2 leaves no deferred push for it to pay; the `rerun` pair's failure on every resume ends.
- The hub workspace file is regenerated after each teardown, so a torn-down pair leaves VS Code.

Bound: an orphan in use stays while it commits or runs a process; an idle one goes after the grace, its branches kept under archive tags.
A kept pair's batten record keeps it out of the orphans, also when its board entry is removed.

Rationale: a bench pair, a kept pair and a trial pair stood for days, listed in the operator's VS Code, with nothing reporting them (A14).
A kept pair stays because keeping it was the operator's explicit choice at the cancel.

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

- D4: an adopter's takeover push rejected over the old owner's later transition writes the takeover entry onto origin's tip and lands, and the adopter reads the run at that transition.
- D4: a second adopt with `--take-hold` landing between an adopter's step 2 and step 3 stops the first adopter at its takeover retry, naming the second's machine, and the second adopter drives the runs.
- D7: a standing `adopt` pause entry from before the upgrade becomes a pause intent on its run, which stays paused; a standing `owner` entry is dropped, and no code path writes either caller after the upgrade.
- D7: a standing `adopt` pause entry from before the upgrade writes one Bolt `adopt` hold naming this machine and posts one notice naming the hold, its machine and a re-run of `lym adopt` there, and no notice names `--take-hold`; a landed run's standing entry is dropped, an upgrade whose standing entries all belong to landed runs or runs owned elsewhere writes no hold, one with a released queued task's entry writes a hold naming that task's subject, and a second machine's upgrade under a standing hold writes no hold.
- D8: `lym adopt <beam>` from a second machine's hub, with origin as the only shared state, ends with the beam's runs in the same FSM states as on the first machine and this machine as owner, with no `--keep-local` or `--drop-local`.
- D8: an adopt run on a machine with no local pair of a moved run adds the pair from origin before it writes the takeover entry.
- D8: two machines adopting at once end with one owner; the other's adopt refuses naming the winner.
- D8: a failing adopt check files one fix task, leaves an `adopt-fix` hold naming it, and exits non-zero; the task's green re-check clears the hold and the runs go on with no hand lift.
- D8: `lym adopt --take-hold` of the hold's subjects on another machine rewrites the hold to name that machine, and the hold clears only at that adopt's green check or, under `adopt-fix`, the fix task's green re-check; with other subjects it refuses naming the hold's subjects; an agent's shell is denied it.
- D8: a hold written by a machine that then stops is taken over by `lym adopt --take-hold` on another machine, whose adopt then runs its steps and check.
- D8: an adopt killed after its owner write goes on from the tracked state on its re-run and ends as an uninterrupted adopt does.
- D8: a transition the old owner pushed between the adopter's fetch and its takeover is on the adopter's local branches before the run acts, and the run goes on from it.
- D8: an adopter whose hold another machine took over stops at its next hold re-read, naming that machine, and starts no dobby.
- D8: a registration D5 refuses at step 1 refuses the adopt with D5's way forward, and the adopt goes on after it.
- D11: a cancelled run's pair without keep is torn down by the dobby; a kept pair stays and is listed with its age in `lym beam status`, also after its board entry is removed.
- D11: an orphan with no commit and no live process is torn down no earlier than the orphan grace after its notice, its branches archived; one with a commit inside the grace stays.
- D11: `lym tender resume` pushes no pair.

## PATTERN changes

- `PATTERN-dobby-runs` — changes: Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt holds the hub through Bolt's holds until an adopt's check is green. — added after Discussion (arbiter, 2026-11-15)
