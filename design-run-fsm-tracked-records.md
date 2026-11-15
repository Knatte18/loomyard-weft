# One run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked

`run-fsm-tracked-records` · Task · feature, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle · running · Drift-Verify

Decision record: [Sketch](decision-run-fsm-tracked-records.md)

Filed from the Sketch `adopt-redesign`.

Takes: shedengine.Status as the batten and child run record, boardengine's run status helpers, battencli's cancel marker, tenderengine's pause and resume, pullrequest's record and landingshed's fixer attempt counts, all as they stand; PATTERN-test-only-code holds, so every new function has a production caller
Delivers: internal/runfsm deriving every run state of D1 from tracked records alone and answering the act rule from its inputs, with the owner, hold, owed-push and code-tip inputs passed in by callers; the dobby's pass, lym batten run, lym loom resume and lym tender resume asking it; the cancel intent with its finish and the run pause intent in the batten record, committed and pushed with today's pushes; a cancelled run's record deleted with its pair; the PR record and fixer counts in the pair's records; the origin-owed marker stays until bolt-holds-catch-up
Exports: runfsm's State type and order, its Derive over tracked records and its Act rule with an inputs struct that later parts fill; the batten record's cancel and pause intent fields and their writers under shedrun's tracked paths

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
