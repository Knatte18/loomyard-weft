# Support log — adopt-redesign

## Interview

The session ran autonomously: no operator answered, and every pick below is an auto-pick recorded in the question ledger.
No advisors were seated.

### Turn 1 — Board and state map

Read the entry `adopt-redesign`, its project `adopt-revision`, the notes `machine-restart-shortcut`, `hub-stop-button`, `per-machine-branch-prefix`,
the draft `claude-account-pick-simple`, and `lym loom peers` (one peer, `testmap-payoff-measure`, no overlap).
No `Decision for adopt-redesign:` note stands.
Read the state map `docs/research/loom/lym-state-map-2026-11-15.md` whole and the PATTERN backgrounds of owner-machine, dobby-runs, lym-files, run-records, shed-driver and pair-remote.

### Turn 2 — Exploration by six subagents

- Machine name: one source, `machinename.Name`, `machine.json` before the hostname; no transform makes `osl46` from `OSL-1046`.
  `osl46` is a name set by `lym machine name`; former names live only in the local `machine.json`, and `OwnedElsewhere` is an exact match. Basis of D5.
- Orch resume: `launchOrch` falls back to a fresh launch on any resume error; `CheckResume` refuses a live holder in `~/.claude/sessions`.
  `BeamUp` reuses a live orch strand only; a Claude process outside a live strand collides. The handover removes the strand but nothing proves the Claude process ends. Basis of D9.
- Push path: the status commit is synchronous, the push a detached child; a failure records a failed-push file and a notice, and only the next push retries.
  No push-all verb exists. Basis of D2.
- Sleep: `pauserecord.SleepWatch` in the dobby loop owes full passes after a sleep; hibernate reconciles on wake. The owner, adopt and legacy hold pause callers and their lifts. Basis of D6 and D7.
- Leftover pairs: `tearDownEnded` covers done or gone entries only; a cancel without `--remove` keeps the pair; a pair with no entry reads `level` and nothing removes it;
  `lym tender resume` pushes every covered pair, the likely `rerun` failure. Basis of D11.
- `.lym`-only records: which carry decisions (cancel, pull request review state, fixer attempts, origin-owed) and which are caches; Finalize searches GitHub for an open pull request before opening one. Basis of D3.

### Turn 3 — Facts checked in this session

- `claude --help` on Claude Code 2.1.325: `--fork-session` makes a new id when resuming, so `--resume <id>` keeps it.
- The prime's transcript directory on `neve`: one session at 206 MB, the two recorded adopt sessions at 9 and 10 MB.
- `pattern.MaxEntryTextChars` is 280, the cap the PATTERN changes keep under.

### Turn 4 — Design picks

Settled D1 to D11 as the question ledger records, each against the PATTERN entries in the record's Constraints.

### Turn 5 — Lens review

Six forks reported: interactions, existing-paths, bound-breaker, walkthrough, board-scope and leanness; none returned no report.
Their reports are in `support-log.lens-<lens>.md`. Duplicates across lenses are merged into one line each.

- D4 judged by commit author breaks on shared records branches (`main-weft`), stops the adopter itself, and treats a foreign pause as a takeover (interactions 1, 2; walkthrough 2, 3; bound-breaker 1, 2; existing-paths `main-weft`): fixed, D4 now reads a takeover entry in the run's own record; the trailer is gone.
- D8 wrote the takeover before the pairs it needs existed and before the stale local tips were reset (walkthrough 1; interactions 2): fixed, step 2 now fetches, archives, resets and adds pairs before step 3's takeover.
- Commits without a trailer and trailer names against the registry (bound-breaker 3; interactions 4; existing-paths D4/D5; walkthrough 10): fixed by dropping the trailer; D5 now routes every machine-name comparison through the registry.
- A Finalize past its parent-side merge killed or removed by D6 and D9 (existing-paths BLOCKING; interactions 7): fixed, D1's act rule, D6 step 4 and D9 carve it out.
- Other writers of `owner` pause entries, the hibernate wake and tender's owned-elsewhere pause (existing-paths BLOCKING): fixed, D7 retires every writer and names what each becomes.
- Holds and pause intents no longer reached the network gate and turn interruption (existing-paths BLOCKING; walkthrough 5): fixed, D3 and D7 keep the local pause record as the enforcement cache, written from tracked intents and Bolt holds.
- Hibernate's pause would become a Bolt hold (bound-breaker 7): fixed, the hub-wide tender pause and hibernate stay machine-local, and Bolt holds only adopt kinds.
- One `hold` key for several holds (interactions 5; walkthrough 4): fixed, `holds` is a list, and a second adopt refuses over another machine's hold.
- The hold covered only the subjects while the operator said the hub (walkthrough 6; bound-breaker 8): fixed, a hold covers the whole hub.
- Readers of the retired adopt record, the dobby handover's and orch watcher's adopt checks (existing-paths MEDIUM): fixed, they read a hold this machine wrote.
- D2 and D3 disagreed on where intents live and who commits them (existing-paths MEDIUM): fixed, intents are in the batten record, committed and pushed by their verb; transitions stage the moved files.
- Tender resume's deferred pushes (existing-paths MEDIUM; interactions 8): fixed, D2 retires fabric's pause deferral, and D11 states the reason.
- Pause or cancel from a non-owner machine (bound-breaker 2): fixed, D3 refuses it naming the owner.
- D1 claimed tracked-only inputs for the act rule (interactions 3): fixed, the state is tracked-only and the act rule names its two local inputs.
- `landed` read from a lagging board status (bound-breaker 12): fixed, `landed` reads the batten record and the landing's board `done` pushes synchronously.
- D1's states overlapped (walkthrough 11): fixed, the states have a precedence order.
- D1's child record absent when the pair is (walkthrough D1): fixed, the record is read from origin's branch then.
- D6 step 4 removed the pair through a full teardown that deletes origin's branch (walkthrough 9): fixed, the removal is local only.
- Uncommitted files neither archived nor removable (bound-breaker 5): fixed, the archive commits them onto the archived tip in D4, D6 and D11.
- A hand pass after an absence skipped the catch-up (bound-breaker 6): fixed, a hand pass is an absence trigger.
- D9's holder test missed the A6/A7 holder, which runproc never registered (walkthrough 7; bound-breaker 4): fixed, the orch's Claude process registers in runproc under an orch role that outlives its strand.
- D10 chose a copy by length, and a stale machine could force-push its mirror (walkthrough 8; bound-breaker 10, 11; interactions 9): fixed, the mirror names its machine, origin's wins when another machine wrote it, and a mirror runs only while owner under a lease.
- The mirror at handover killed by D9's ends (interactions 6): fixed, no mirror runs at handover, and one killed mid-push leaves the ref unchanged.
- D10's session record had no commit path (existing-paths LOW): fixed, the orch start commits and pushes it.
- D11's teardown condition read the retired cancel marker and pause record (existing-paths LOW): fixed, it reads the FSM state and the act rule.
- D11's grace could be zero, and a kept pair with its entry removed read as an orphan (walkthrough 12; bound-breaker 13): fixed, the grace runs from the notice, and an orphan has no batten record either.
- D5 left the former names of `machine.json` out, and a reused name (walkthrough 10; bound-breaker 14): fixed, the first registration copies them and a clash is refused.
- `--release` bypassable by unsetting the strand variable (bound-breaker 9): fixed, a shell-guard row denies it to agents, and no refusal an agent reads names it, per `PATTERN-refusal-way-forward`.
- `--release` against "until every failing test is fixed", and the orch up during a hold (board-scope 1, 2, class decision): kept as picked and raised under Open risks for the parent.
- Kept pairs never torn down (board-scope 3): rejected, keeping the pair was the operator's explicit choice at the cancel; D11 reports them, and its rationale now says so.
- D3's pull-request criterion held only through GitHub's search (bound-breaker 15): fixed, the record commits and pushes when written.
- Scope's In list restated the Decisions (leanness): fixed, it points to them.
- The process note under Open risks and the D10 deferred manual check (leanness): fixed, both dropped; D10's fact cites the resume check every orch resume passes today.
- Auto-mode assumptions restating Decisions, and Constraints repeating them (leanness): fixed, trimmed to what no Decision states.
- The Goal's bug list and D1's consumer lists (leanness): fixed, cut to a pointer and a rule.

## Rejected alternatives

- A `Lym-Machine` commit trailer, judging a rejected push by who committed: rejected for D4 after the lens review; records branches such as `main-weft` hold many runs and non-owner commits, so a foreign commit is no takeover.

- A new run state file beside `status.json`: rejected for D1, since the batten and loom records already hold the phase machine; one package deriving over them adds no second writer.
- Keeping the detached per-transition push and adding a background retry: rejected for D2; the state would still lag origin with the run acting on it.
- Letting a run act offline and pushing later: rejected for D2; another machine could not go on from a state it never saw.
- Fetching Bolt's owner before each transition: rejected for D4; a fast-forward-only push on the run's own branch excludes a second machine with no extra round trip.
- Normalising machine names, such as stripping dashes: rejected for D5; `osl46` was a chosen name, not a spelling of the hostname.
- Keeping `--keep-local` and `--drop-local`: rejected for D8; with D2 and D4 the only local-only commits are this machine's owed pushes or work another machine's takeover archives.
- Starting the orch only after the hold clears: rejected for D8; the fix task's halts and notices need an orch to read them.
- Committing the orch's transcript on the weft branch: rejected for D10; each mirror would add a full copy to the branch history, and a 206 MB transcript breaks GitHub's file limit.
- Compressing the transcript mirror: rejected for D10; compressed chunks delta poorly, so each mirror would upload the whole file.
- Carrying every agent session's transcript: rejected for D10; a run's step starts again from its pushed transition, so only the orch needs continuity.
- Removing entry-less pairs from the workspace file but leaving them standing: rejected for D11; the operator asked that they not stand.

## Review rounds

### Round 2

- A Finalize past its parent-side merge now hands over after a takeover like any run, and its owner finishes it (D1, D4, D6, D9); Turn 5's carve-out on the old owner is gone.
- D10 restores the orch's transcript by the session record's last launcher or resumer, no longer by the machine the mirror names.
- A pair's transition pushes its records side before its code side, and each records commit names its code tip (D2, D4).

## Question ledger

- Q1, auto-pick: where the FSM lives. Picked a derivation package over the tracked batten and loom records and Bolt (D1), since those already hold the phase machine.
- Q2, auto-pick: whether a run may act offline. Picked no (D2): the operator requires the state on GitHub at all times. Raised under Open risks.
- Q3, auto-pick: how two machines are kept from driving one run. First picked a machine trailer judged on rejection; after the lens review, a takeover entry in the run's own record, read after a replay (D4).
- Q4, auto-pick: the cause and fix of A8. Picked a Bolt machine registry of former names (D5).
- Q5, auto-pick: what counts as an absence. Picked an observed sleep, a dobby start and a hand pass (D6), reusing `pauserecord.SleepTolerance`.
- Q6, auto-pick: what a catch-up does with this machine's commits on a run owned here. Picked pushing them as owed transitions (D6).
- Q7, auto-pick: what happens to the local pair of a run owned elsewhere. Picked removal after archiving its tips (D6).
- Q8, auto-pick: where the hold lives and which pauses it replaces. Picked a Bolt `holds` list of adopt kinds replacing the owner and adopt pauses; the hub-wide operator pause and hibernate stay local, and the local pause record stays the enforcement cache (D7).
- Q8a, auto-pick: whether a hold covers its subjects or the hub. Picked the hub, per the operator's "nothing else happens in the hub" (D7).
- Q8b, auto-pick: whether a non-owner machine may pause or cancel a run. Picked no, refused naming the owner, so only the owner writes a run's records (D3).
- Q12a, auto-pick: how lym knows a session holder is its own orch. Picked a runproc registration under an orch role that outlives the strand (D9).
- Q9, auto-pick: whether runs go on without a hand lift after adopt. Picked yes once the check is green (D8), changing `PATTERN-dobby-runs`; existing adopt pauses convert to run pause intents.
- Q10, auto-pick: whether the orch is up during a fix hold. Picked yes, with run verbs refused (D8).
- Q11, auto-pick: an escape from a check that cannot pass on this machine. Picked `lym adopt --release`, refused from agent strands (D8).
- Q12, auto-pick: what to do with a live holder of the orch's session. Picked ending it when lym owns it, refusing otherwise, never launching fresh over it (D9).
- Q13, auto-pick: whether the orch's transcript is pushed to GitHub. Picked yes, through a replaced ref in the weft repository (D10). Raised under Open risks.
- Q14, auto-pick: the mirror interval and the orphan grace. Picked 10 minutes and 24 hours (D10, D11).
- Q15, auto-pick: whether kept pairs are torn down. Picked no, listed and noticed only, since the operator chose to keep them (D11).
