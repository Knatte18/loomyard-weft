# One run FSM over tracked records, with cancel and pause intents

A run's lifecycle state is now derived by one package, `internal/runfsm`, from tracked records alone.
Before, six vocabularies disagreed on `paused`, `running`, `done` and the halted states, and a run's cancel and pause decisions lived only under a machine's `.lym`.
Now every machine reads the same state from the same pushed bytes, and a run whose `.lym` is removed reads the same state as before.

## What changes

- `internal/runfsm` declares the states in precedence order: `landed`, `cancelled`, `cancelling`, `paused`, `halted`, `landing`, `running`, `starting`, `queued`.
  `Derive` reads the board entry, the batten record and the loom child run record, from the local worktree or from origin's branch when this machine has no pair for the run.
- The act rule answers `act`, `wait` or `hand_over` from the run's state and from inputs its callers pass in: the owner, the Bolt holds, an owed push with origin unreachable, the code tip origin must hold, and whether origin's fetched parent branch puts a `landing` run past its parent-side merge.
  The dobby's pass, `lym batten run`, `lym batten step`, `lym loom resume` and `lym tender resume` ask it first and refuse on `wait` or `hand_over`, naming the way forward.
  A pause intent, a cancel intent, a hold and a shed state `paused` are asides for the verbs that resolve them, so `lym tender resume` and `lym batten cancel` can act on a paused, held or half-cancelled run.
- The cancel intent and its finish live in the batten record in place of the cancel marker file, which is gone.
  `lym batten cancel` commits and pushes the intent before it ends anything, then writes the finish, the time it ended and remove or keep.
  A finished cancel keeps the record with the shed state `cancelled`, so the dobby's done-run clear never takes it.
  Every pair removal (`--remove`, the dobby teardown and `lym fabric remove`) deletes the record in the same step.
  `lym batten cleanup` only repairs a leftover: it removes a `cancelled` record whose pair is gone and refuses one whose pair stands, naming `lym fabric remove <slug>`.
- `lym tender pause <slug>` writes a run-scope pause intent into the batten record and `lym tender resume <slug>` clears it, each committing and pushing before returning.
  Each still writes this machine's pause record entry.
  Either verb on a run another machine owns is refused, naming that machine, with the same verb there or `lym adopt` here first as the way forward.
- The pull-request record (`internal/pullrequest`) and the fixer attempt counts (`internal/landingshed`) are written into the pair's tracked records, and each write commits and pushes.
- `internal/shedrun` gains the intent writers under its tracked paths, `internal/fabricengine` gains reads of origin's fetched refs, and `internal/pairteardown` gains `ReadRun` as the one reader of a run's FSM inputs.
  The board's `<state> · <row>` status is now written from the FSM state.
- `PATTERN-lym-files` now says `.lym` holds only caches a fresh machine rebuilds or starts without.

## Design decisions

- No run acts while origin is unreachable: an owed push holds the run at its transition until it lands.
  Offline, `lym tender pause`, `lym tender resume` and `lym batten cancel` act on this machine at once and owe their push; until it lands, another machine reads the run unpaused or not cancelled.
  A takeover the owed push meets drops the intent, and the notice names each dropped intent.
- A `cancelled` record whose task branch is gone from origin, which a crash between a pair removal and the record's deletion leaves, counts as no record, so the released task reads `queued`.
  A kept pair's record reads `cancelled` on every machine.
- A `landing` run is past its parent-side merge only when origin's fetched parent branch holds the task's work, never the local parent branch.
  Under the push skip the local parent branch decides it.
- The origin-owed marker of `internal/battencli/resumereconcile.go` stays until the Bolt holds catch-up part replaces it.
- Tracked values that hold only on the writing machine, such as Claude session ids and absolute paths in step envelopes, are read as such: a session id with no local transcript starts that session fresh.

## To check by hand

- A cancel's intent can be invisible to other machines before it is committed and pushed; check that a cancel interrupted before the push reads sensibly from the owner machine and that `lym batten cancel <slug>` run again finishes it.
- `readrun`'s past-parent-merge read on a machine with no pair for the run: check that a `landing` run reads from origin's refs alone, with no local worktree to fall back on.
