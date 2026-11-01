# Fixer report — discussion round 1

Every finding's premise held against the code.
All findings are fixed in `_lym/discussion/decision-record.md`; the support log's Review rounds section records the round.

## Changes

- **F1 (BLOCKING), as proposed.**
  D1: the runner claims its subdirectory with a process file (pid and start time) at start, and a front joins only a run that has an unmarked exit file or a live runner with the recorded start time; otherwise it supersedes.
  D3: a terminating signal to the runner records 128 plus the signal number; a runner killed without an exit file frees its slots through its OS file locks (`internal/gateslot/doc.go`) and the next front supersedes it, with the bound for a front already waiting.
  New D1 criterion for a killed runner.
- **F2 (MEDIUM), as proposed.**
  D3: a front writes its deadline only when later than the lease, under the key lock, and the supersede expiry is the one write that moves it earlier; the Bound now reads the latest deadline.
  New D3 criterion for a short-cap front joining a long-cap front's run.
- **F3 (MEDIUM), fixed differently.**
  D5 records that today's `RunRecord` has no cancel mark and that `execution.report` writes a cut tier with its exit code, adds `Abandoned` (new), defines a call's wall time, and states that a refused call drops out by itself.
  Not made: the runner setting the mark on every record of the call.
  Records of tiers that ended before the cancel are already written when the cancel comes (`internal/gatecli/run.go` `report` writes each tier's record as it ends), so marking them would mean rewriting records.
  Instead `execution.report` sets `Abandoned` on a record it writes after the call's context was cancelled, which also covers an in-process call a signal ended, and the median leaves out a call any of whose records carries it.
- **F4 (MEDIUM), as proposed.**
  D6 states each line kind from `internal/cardverify/runner.go`: a `--wait` gate line runs as a child taking its own slots, with no `slot_free` answer; any other line takes one slot inside the runner.
  `cardverify.Runner` takes the caller's context in place of `context.Background()`, and the cancel reaches the line through `verifyrun.Run`'s process-group terminate.
  The claim that the slot is always taken inside the runner is gone; new D6 criterion for the cancel.
- **F5 (MEDIUM), as proposed, first option.**
  D1: the runner verb refuses before it runs anything when its subdirectory holds no lease, a passed lease or a process file, naming `lym gate test` and `lym gate card-verify`; the Bound states what a direct call can reach.
  New D1 criterion for a direct call.
- **F6 (LOW), fixed differently.**
  D3: a starting front writes the lease before it spawns the runner.
  Not made: "a runner finding no lease ends as abandoned".
  The F5 refusal already covers a runner that finds no lease, and refusing before anything runs writes no exit file, so no front reads a missing lease as an abandoned run.
- **F7 (LOW), as proposed, first option.**
  Constraints: every `verifyrun.Run` caller removes the strand, naming `internal/battencli/land.go`'s `runDeploy` and `runRowCommand`, which pass a nil environment today; Scope In gains the line.
- **F8 (LOW), as proposed.** D1's Bound states the second front's fresh run.
- **F9 (LOW), as proposed.** D1's Bound states the 75 exception.
- **F10 (LOW), as proposed.** D9's bullet and criterion name the `shuttleengine.DefaultAwaitCap` doc comment in place of the stencils.
- **F11 (LOW), as proposed.**
  D4's Rationale cites the settings text built into Claude Code 2.1.316: the subagent lifetime is "Unset = automatic (5 minutes unless ENABLE_PROMPT_CACHING_1H=1)", with `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` taking precedence, and the claude engine sets neither (`internal/shuttleengine/claudeengine/doc.go`).
  The source is the installed binary's text, not a documentation page; none was at hand.

## Self-check

- fix-consistency, MEDIUM, card-verify strand strip covers only the verb, not `cardverify.Runner` under a Go verb: fixed as proposed; D2 now has `cardverify.Runner` strip the strand for every line, and the Constraint names it.
- fix-consistency, LOW, D3 Rationale still says one cap plus the grace: fixed as proposed (the latest front's cap).
- fix-consistency, LOW, tolerant-records Constraint and D5 criterion name only the call id: fixed as proposed (`Abandoned` added to both).
- fix-consistency and fix-claims, LOW, D6 "which D7 keeps for every card": merged; fixed as proposed ("the form D7 keeps for a card's gate call").
- fix-consistency and fix-way-forward, LOW, the claim wait is unbounded and missing from PATTERN-daemon-wakeups: merged; fixed with a once-a-second check for at most 30 s, after which the front expires the lease so a late claim is refused and exits 1; the Constraint and a D1 criterion name it.
- fix-way-forward and fix-proposal, MEDIUM, a signal-ended runner's 143 replays as a failed gate: merged; fixed differently.
  Not made: an added stderr line on the replayed 143, since an agent still reads a non-75 exit as a failure.
  The exit file of a run that a lease, a supersede or a signal ended carries an abandoned mark; a front never replays it, answers 75 with `done: false`, and the next call joins or starts the current run (D1 bullet, D3, D3 criterion).
- fix-proposal, LOW, a superseded run's exit 1 reaches a front still waiting on it: fixed by the same abandoned mark.
- fix-way-forward, LOW, the refusal promises the same words the runner may not know: fixed as proposed; it names `lym gate test` and `lym gate card-verify` without words.
- fix-claims and fix-proposal, MEDIUM, a build step's record has a zero `WaitStart`, so the earliest one spans millennia: merged; fixed as proposed; a record counts from `Start` where `WaitStart` is zero (`internal/gatecli/build.go`), in D5 and its criterion.
- fix-claims, MEDIUM, a role cap above 55m breaks D3's bound: fixed differently; D4's validation also requires each cap to be at most the long cap, so the bound holds and a cap never outlasts the long wait the Bash ceiling sizes.
- fix-claims, LOW, `runRowCommand` is a batten recipe row's command, not a landing config's: fixed as proposed.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
