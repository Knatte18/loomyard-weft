# Support log — dbench-ohsh

## Interview

Autonomous mode: no operator turns.
The chair read the board entry, the related entries (`weaver`, `proc-child-lifetime`, `gate-test-hygiene`), the seven linked issues and the code at HEAD `800a0e892`.
It then sent two question batches to ly:dbench-ohsh:discussion-advisor-1.

### Batch 1 — items 1–3

- Chair's findings:
  - Commit `5a8be0274` (#496) already records ended shells (warning, note, summary).
  - A webster `done` requires the verify gate, which includes the integration tier.
  - Shuttle's `gateRepromptText` says "fix every finding", contradicting Master's stencil.
  - Recovery sessions are absent from `recordedSessions`.
  - `recover-batch` returns `ok` for a stuck or dead digest.
- Q1, gate shells. Chair's recommendation: the gate re-prompt fix, a Master stencil rule never to run tests, and the reworded record.
  ly:dbench-ohsh:discussion-advisor-1 agreed with the stencil rule and the rewording, but corrected the root cause:
  - Master started the shells before any gate message.
  - `countsAtOnce` ends a fresh gated run at once on unawaited shells, so the stencil rule, not the re-prompt fix, is item 1's fix.
  - lyx kills no shell (`logEndedShells`), so the record must say the shell "ended with the strand".
  - The advisor also named the stale prose to update.
- Q2, Master self-fix. Chair's recommendation: a per-entry re-prompt text on `GateEntry`, with webster's text routing to the fixer fork.
  ly:dbench-ohsh:discussion-advisor-1 agreed:
  - It confirmed the call site in `wait.go` and the one-line `validateSendText` rule.
  - It found that `renderVerifyGateFindings` also contradicts the stencil ("Commit or remove these paths"), and that the verify-fix body already covers a dirty tree.
- Q3, recovery attribution. Chair's recommendation: record recovery session ids on the batch and include them in `recordedSessions`.
  ly:dbench-ohsh:discussion-advisor-1 agreed, with three corrections:
  - route the recovery writes to `RunWrites.Forks` because of `contractevidence.go`;
  - skip a recovery session whose transcript is missing instead of failing `loadRunWrites`;
  - carry the id from the existing `FindRun` call across respawns, `omitempty`.
- Q4, exit code. Chair's recommendation: non-zero on any non-done terminal digest.
  ly:dbench-ohsh:discussion-advisor-1 agreed with the direction, noted three points, and suggested asking the operator for the envelope:
  - `failed` already exits non-zero, so the board's "failed" is unverified;
  - the change makes `recover-batch` differ from `record-batch`;
  - it adds friction notes.
  No operator in autonomous mode, so this became an auto-pick.

### Batch 2 — items 4–6 and scope edges

- Chair's findings:
  - Quarry resolves unexported and `_test.go` members `found`.
  - `internal/proc#StartTime` resolves `ambiguous`.
  - Quarry is an external module.
  - Plans compile the tmux tier, and Publish runs it.
- Q5, recovery gates. Chair's recommendation: state the allowed gates and the tmux rule, leaning toward the shared body.
  ly:dbench-ohsh:discussion-advisor-1 agreed and put it all in `webster-body-implementer.md`:
  - The fabric ban lives there.
  - The tmux sentence must sit next to step 5, since a card's `verify:` fails the card.
  - The text says no report is `FAILED` over a compile-only step.
  - `go vet -tags tmux <pkg>` is not denied, but `./...` and a raw `go test -tags tmux` are.
- Q6, refusal text. Chair's recommendation: name the lookup and quarry's answer.
  ly:dbench-ohsh:discussion-advisor-1 agreed:
  - Build the text where the finding is built, so the plan gate and `batch_failed` share it.
  - An ambiguous Create's way forward is a plan edit.
  - A refusal-spec row is needed.
- Q7, build variants. Chair's recommendation: one predicate, called in each consumer.
  ly:dbench-ohsh:discussion-advisor-1 agreed with the predicate and proposed applying it once at `resolveTargets` (and the exported `Resolve`) by rewriting a variant-set `ambiguous` to `multipart`. Its reasons:
  - every consumer already accepts `multipart`;
  - the status tests stay untouched;
  - `lyx quarry resolve` agrees with the gate;
  - `handle.go` keeps rejecting `multipart` for a rename signature.
  Soundness note: `go/build` has no exported GOOS/GOARCH list, so use a fixed platform set and a tag cap, failing safe.
  The chair adopted the chokepoint form.
- Q8, scope edges. ly:dbench-ohsh:discussion-advisor-1 agreed to keep all three out:
  - #507's delete-and-recreate becomes a documented residual;
  - `reset --to batch-start` covers the dead-batch revival;
  - nothing needs to move.
- Testing. ly:dbench-ohsh:discussion-advisor-1:
  - counted the budget rows as tight in four packages;
  - advised rows in existing tests and listing the budget file under `Edit:` for any new top-level test;
  - flagged the collision risk with `gate-test-hygiene`;
  - confirmed that nothing needs the integration, tmux or llm tier.

## Rejected alternatives

- Item 1:
  - Hold Master's turn end while a `lyx gate test` shell runs, by adding it to `AwaitedShellPrefixes`. Rejected: its result cannot change the outcome because the verify gate re-runs the tests; it adds wall time and competes for gate slots; and the prefix match on the command string misses `cd x && …` forms.
  - Demote a run to stuck when a gate shell is outstanding. Rejected: it punishes a run whose verify gate passed.
  - lyx kills the shell explicitly. Rejected: shuttle holds no pid for a Bash-tool child and has no provider-neutral way to reach it.
- Item 2:
  - A webster verb that spawns the fixer. Rejected: the fixer is an in-session Agent-tool fork, which Go cannot create; a cold fixer strand is a Weaver-sized design.
  - Stencil-only rewording. Rejected: the message Master acts on would still contradict it.
- Item 3:
  - Start the second recovery from a clean tree with the first attempt's diff as reference. Rejected: it needs a destructive reset through the fabric destruction chokepoint and discards work.
  - Carry the first spawn's foreign set and treat later dirty paths as own. Rejected: it guesses without transcript evidence and would misattribute strays.
  - Recovery writes as `RunWrites.Master`. Rejected: it changes `contractevidence.go`'s fork-after-Master reading.
- Exit code: non-zero only when `recovery_retry` is false. Rejected: inconsistent, and it leaves `stuck` exiting 0.
- Item 4: the gate rules in the recovery prefix only. Rejected: the ambiguity and step 5 live in the shared body, and forks face the same rule.
- Item 6:
  - Make quarry honour build constraints. Rejected: it is an external repository, and quarry deliberately answers without a build context.
  - Accept any `ambiguous` answer at DoneChecks. Rejected: it hides genuine duplicates.
  - Call the predicate in each consumer. Rejected in favour of the chokepoint rewrite: it threads the root through five consumers and trips the status tests.
  - Allow `create-already-exists` to pass when the same card deletes every declaration. Rejected as unneeded once variant sets resolve.
- #511 point 3, `goto` revives a dead batch. Rejected: `reset --to batch-start` exists, and the item is not in the board body.

## Review rounds

_No rounds yet._

## Question ledger

- Q1, gate shells: resolved, advisor-informed (ly:dbench-ohsh:discussion-advisor-1). A stencil rule plus the reworded record; no wait change; no kill.
- Q2, gate re-prompt: resolved, advisor-informed (ly:dbench-ohsh:discussion-advisor-1). A per-entry `GateEntry` text, and the same rule at the top of the findings file.
- Q3, recovery attribution: resolved, advisor-informed (ly:dbench-ohsh:discussion-advisor-1). Session ids on the batch, routed to Forks, a missing transcript skipped.
- Q4, recover-batch exit: auto-pick. The advisor suggested asking for the quick-recipe envelope, and no operator answers here.
  The chair picked non-zero for terminal `stuck` and `dead`, reading the board's "failed" as one of those, because `failed` already exits non-zero.
- Q5, recovery gates: resolved, advisor-informed (ly:dbench-ohsh:discussion-advisor-1). Both statements go in the shared implementer body.
- Q6, refusal text: resolved, advisor-informed (ly:dbench-ohsh:discussion-advisor-1). The text is built at finding construction.
- Q7, build variants: resolved, advisor-informed (ly:dbench-ohsh:discussion-advisor-1). A chokepoint rewrite to `multipart`, a fixed platform set and a tag cap.
- Q7a, the exact platform list and tag cap: auto-pick, left to the plan writer within D8's bound.
- Q8, scope edges: resolved, advisor-informed (ly:dbench-ohsh:discussion-advisor-1). All three stay out.
- Q9, test budget: resolved, advisor-informed (ly:dbench-ohsh:discussion-advisor-1). Rows in existing tests; the budget file goes under `Edit:` for any new top-level test.
