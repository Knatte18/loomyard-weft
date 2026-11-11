# Card 7 — adopt-worker

**Create:**
- `plan:internal/adoptcli#workerDeps` -> `type workerDeps struct`
- `plan:internal/adoptcli#hubWorkerDeps` -> `func hubWorkerDeps(prime *lymcwd.Location, machine func() (string, error)) (workerDeps, error)`
- `plan:internal/adoptcli#runWorker` -> `func runWorker(deps workerDeps, req request) (outcome, adoptrecord.Record, error)`
- `plan:internal/adoptcli#validateRequest` -> `func validateRequest(deps adoptDeps, req request) (slugs []string, root bool, err error)`
- `plan:internal/adoptcli#standingVerdict` -> `type standingVerdict string`
- `plan:internal/adoptcli#verdictFresh` -> `const verdictFresh standingVerdict = "fresh"`
- `plan:internal/adoptcli#verdictFollow` -> `const verdictFollow standingVerdict = "follow"`
- `plan:internal/adoptcli#verdictResume` -> `const verdictResume standingVerdict = "resume"`
- `plan:internal/adoptcli#judgeStanding` -> `func judgeStanding(record adoptrecord.Record, found bool, state string, call adoptrecord.Request) (standingVerdict, error)`
- `plan:internal/adoptcli#mergeRequest` -> `func mergeRequest(recorded, call adoptrecord.Request) (merged adoptrecord.Request, restart bool)`
- `plan:internal/adoptcli#request.recorded` -> `func (r request) recorded() adoptrecord.Request`
- `plan:internal/adoptcli#requestOf` -> `func requestOf(recorded adoptrecord.Request) request`
- `plan:internal/adoptcli#envelopeOf` -> `func envelopeOf(record adoptrecord.Record) (outcome, error)`
- `plan:internal/adoptcli#workerCmd` -> `func workerCmd() *cobra.Command`
- `plan:internal/adoptcli#world.workerDeps` -> `func (w *world) workerDeps() workerDeps`
- `plan:internal/adoptcli#TestRunWorker` -> `func TestRunWorker(t *testing.T)`

**Delete:**
- `internal/adoptcli#adopt`

**Edit:**
- `internal/adoptcli#Command`
- `internal/adoptcli#namedEntries`
- `internal/adoptcli#AdoptAs`
- `internal/adoptcli#world`
- `internal/adoptcli#newWorld`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver`

**Uses:**
- `internal/adoptcli#stepRun`
- `internal/adoptcli#runSteps`
- `internal/adoptrecord#Read`
- `internal/adoptrecord#Write`
- `internal/adoptrecord#LockPath`
- `internal/adoptrecord#Record.ResumeStep`
- `internal/adoptrecord#Record.ReadState`
- `internal/adoptrecord#Request.SameSubjects`
- `internal/adoptrecord#Request.Command`
- `internal/adoptrecord#WorkerAlive`

**Intent:**
D1's worker and D2's record and resume.
The hidden `lym adopt worker [<subject>...]` verb (`workerCmd`, audience internal) takes `--all`, `--keep-local`, `--drop-local` and `--no-orch`.
It resolves the prime as the adopt does, builds `hubWorkerDeps`, runs `runWorker` and prints the envelope: ok for `done`, an error envelope for a stop or refusal.
`Command` adds it beside `check`.
Like `check`, the word `worker` shadows a beam so named, which `--all` still adopts.

`workerDeps` holds the step deps (`adoptDeps`), the hub path, this process's pid and start ticks, a clock, the record read and write, the non-blocking `adopt.lock` acquire, and the liveness read `adoptrecord.WorkerAlive`.
`hubWorkerDeps` fills them over `hubDepsAs`.
`validateRequest` holds the checks `adopt` ran before any step: the `root` word, `--keep-local` with `--drop-local`, `--all` with slugs, no subject, and `namedEntries` over the owners.
The caller runs it too (card 8).

`runWorker`, in order:
1. `validateRequest`; a refusal returns before any lock or step.
2. It takes `adopt.lock` through `lock.TryAcquireWriteLock` and holds it for the whole run.
   A held lock returns at once with an error naming `lym beam status`, writing nothing.
3. It reads the record and judges it with `judgeStanding` over its `ReadState`.
   - With no record, or a `done` one: verdict fresh.
   - An unfinished record (`running`, `interrupted` or `stopped`) whose request `SameSubjects` the call's: `follow` while its worker reads alive, else `resume`.
     A `follow` verdict here is a transient refusal naming `lym beam status` and `lym adopt --discard`, returned before any step and writing nothing.
     It is reached whenever the record's worker reads alive while this worker holds the lock, as for a record without start ticks whose pid another process now holds.
   - An unfinished record with other subjects: refused.
     The refusal names the record's subjects, the step it stopped at, its `Request.Command`, and `lym adopt --discard`; it writes nothing.
4. Fresh: it writes the record `running` at `reconcile`, with the request, the resolved subjects, its pid, start ticks and start, before any step runs.
   Resume: `mergeRequest` keeps the record's flags and adds the call's.
   A call's `--keep-local` or `--drop-local` replaces the record's other one, and `restart` is set when the call adds either.
   The worker rewrites only the pid, start ticks, start and state, `running`, keeping the done steps and stored inputs.
   It runs from `ResumeStep`, or from `reconcile` on `restart`.
5. It runs `runSteps` with `Save` writing the record, and at the end writes the record with `envelopeOf`'s envelope.
   `envelopeOf` merges the outcome part of every step the record lists as done, an earlier process's steps included, in step order.
   It then merges the record's `StoppedPart`, when a stop left one, without listing that step as done.
   A scalar field of a later part replaces an earlier one; `slugs` merges per slug and per field; list fields concatenate.
   The merged slugs are then rendered as a list in the order `collect` gives, over the record's `Subjects` without `root` that the merged parts hold.
   So a stop before any step sets a slug field, such as a refused reconcile, lists no slugs, as today's reconcile refusal does.
   It returns the outcome, the final record, and the stop's error.

A resumed step past `owner` reads its inputs from `Inputs`, never from the earlier process.
Each step is safe to run again (the record's Constraints): a re-run adds only missing pairs, pauses only slugs not listed, writes the same owners, judges the deploy again, leaves a live pair to the beam steps, and starts no second live check.

What the hidden verb lets through: it is one more entry into the steps, callable from any shell.
It is bounded by the same validation, D2's subject match and `adopt.lock`.
A worker that `lym fabric anchor`'s `runproc.EndSparing` ends mid-step leaves the record `running` with a dead pid, which reads `interrupted`, and a re-run resumes it.
`ReadCheckViews` and `primeMark` skip non-directories, so `adopt.json` and `adopt.lock` beside the subject directories change no check view.

`adopt` is deleted: `Command`'s `RunE` calls `runWorker` in-process over `hubWorkerDeps` until card 8 moves it into a spawned worker.
`AdoptAs` runs `runWorker` over `hubWorkerDeps` with the named machine and returns its outcome.
`namedEntries` keeps its refusals; its doc says `validateRequest` calls it.

Tests, tier0, over the fake world with the record and lock in `t.TempDir()`:
- `TestAdopt` drives `runWorker` through `world.workerDeps` in place of `adopt`.
  Its rows are unchanged otherwise, but three gain assertions on the persisted record:
  - card 6's held-back row reads it `stopped` at `done`, with a way forward naming `--keep-local` and `--drop-local` through `Request.Command`;
  - card 6's failed-deploy row reads it `stopped` at `deploy`, with `owner` among the done steps;
  - `ReconcileErrorRefuses` reads it `stopped` at `reconcile`, with no later step in the order.
    Its faked reconcile returns mutations beside its error, and the envelope keeps the reconcile report and the mutations, with `partial` true.
- `TestRunWorker`, one row per worker behavior the `TestAdopt` rows do not reach:
  - A fresh run's first `Save`, before the world's order holds any step, writes the record `running`, `InFlight` `reconcile`, with the request and the resolved subjects.
  - A record whose steps run through `owner` with a dead pid, re-run with the same subjects, resumes at `deploy`: the world's order shows no `reconcile`, `addpair:`, `pause` or `owners`, and the record ends `done`.
  - One whose steps run through `pairs`, re-run, runs the reconcile again before its owner write, and the record lists each step once.
  - A record listing a slug in `Inputs.Paused` whose entry the operator lifted since, re-run from `reconcile`, leaves that slug with no entry.
  - A held lock returns at once and leaves the record unchanged.
  - Other subjects are refused before any step and leave the record unchanged.
    The refusal names the recorded subjects, the step it stopped at, the re-run and `lym adopt --discard`.
  - A `running` record of the same subjects whose pid and start ticks are the test process's own is refused before any step, naming `lym beam status` and `lym adopt --discard`, and the record stays unchanged.
  - A record done through `owner`, re-run while a move record stands in a subject's worktree, is refused before `deploy` naming the move's way forward: the order holds no `deploy`, and the record ends `stopped` at `deploy`.
  - An invalid request (`--keep-local` with `--drop-local`) is refused before the lock and any step.
  - A resume keeps the record's `--no-orch`, seen on every `BeamUp` call.
  - A resume adding `--keep-local` to a record stopped on a held-back project restarts at `reconcile` and ends `done`, its `HeldBack` cleared.
  - A resume adding `--drop-local` to a record with `--keep-local` records `--drop-local` alone.
  - A done record's envelope merges the parts of steps from two processes: a resume after `owner` still carries the earlier `owner` step's per-slug `owner_recorded`, and the earlier process's reconcile warnings, part of that `owner` step, survive.
- `TestAdoptIntegration_SecondCloneTakesOver` (tier1) also reads the second clone's `adopt.json` as `done` with every step of `adoptrecord.Steps` listed.
  The first clone's adopt back returns an error naming `--keep-local` and `--drop-local`, with the record stopped at `done`.
  The `--drop-local` call resumes it from `reconcile` and ends with the pair-add refusal naming `--keep-local`.
  The `--keep-local` call then records the prime's owner.

**ImpactSummary:** the in-process `adopt` gives way to `runWorker`, whose callers are `Command`, `AdoptAs` and the tests; the record and lock are new files under the hub's adopt scratch directory, and no package outside adoptcli changes.
