# Card 10 — tender-revive

**Edit:**
- `internal/pauseengine#Pair`
- `internal/pauseengine#Deps`
- `internal/pauseengine/doc.go`
- `internal/pauseengine#fakePair`
- `internal/pauseengine#env`
- `internal/pauseengine#env.deps`
- `internal/pausecli#Command`
- `internal/pausecli#refuseCaller`
- `internal/pausecli#hubDeps`
- `internal/pausecli/doc.go`
- `internal/pausecli#TestPauseAndResume_WriteLiftAndRefuseOverTheHub`
- `internal/pausecli#TestPauseAndResume_ApplyTheCallerRules`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/tenderengine#ReviveLine`
- `internal/tenderengine#ReviveRequest`
- `internal/tenderengine#ReviveReport`
- `internal/tenderengine#OrchRevive`
- `internal/tenderengine#Revive`
- `internal/tenderengine#reviveSession`
- `internal/tenderengine#reviveOrch`
- `internal/tenderengine#fakePair.SendConditioned`
- `internal/tenderengine#TestRevive`
- `internal/tendercli#reviveCmd`
- `internal/tendercli#shuttlePair.SendConditioned`

**Intent:**
`lyx tender revive [<slug>]` gives one fixed carry-on line to each session stalled on an API error, the case a network outage without a machine sleep leaves behind (D5).
The engine code goes into `resume.go` and its test into `resume_test.go`, so the files card 12 renames are the ones that exist now.
Both files' header comments then name the revive sequence beside the resume sequence and the status read.

`ReviveLine` is the one fixed line, declared once in `pauseengine`.
`Pair` gains `SendConditioned(guid, text string, want shuttleengine.SendCondition) (turnStarted bool, err error)`, which `shuttlePair` implements over card 4's `Runner.SendConditioned`.
`Deps` gains:
- `OrchStrand func() (string, error)`, the orch strand recorded in the prime's orch state, empty when none;
- `QueueOrchRevive func(line string) (queued bool, err error)`, which queues card 5's conditioned notice and returns false when one is already queued.

`Revive(deps, req)` covers the hub for an empty slug and one pair for a slug.
For each live session of each task pair in scope, in order, `reviveSession`:
1. reads the session's `State`; a failed read is reported refused with the error;
2. skips a session whose state is not idle-stalled with cause `api-error`, reporting it skipped with the state read; it is never waited for;
3. runs, inside `pauserecord.Update` with a mutate that changes no entry, a coverage read of the record and then the conditioned send naming idle-stalled with cause `api-error`.
   The send runs under the record's write lock, the lock `pause` takes to add its entry, and the lock is held for one conditioned send at a time.
   `pauserecord` gains no call and keeps its import set, and the write at release prunes expired resumed entries as every record write does.
4. reports a session a live entry covers at that read as `paused`, with no send, since `resume` gives it its line;
   a send with a turn start seen as `sent`;
   one that landed with none seen as `sent_no_turn_start`;
   `ErrSubmissionNotLanded` as `not_landed` with its reason;
   `ErrSendConditionUnmet` as skipped, the session that turned busy or otherwise changed between the read and the send;
   and any other error, a guard refusal such as a draft, operator activity or the rewind picker, as `refused` with its reason.
   No send is retried.

For the hub scope only, `reviveOrch` then reads `OrchStrand` and the orch's `State` through `Prime`.
No other prime session is read, revived or reported.
An orch whose state is idle-stalled with cause `api-error` is reported `paused` when a hub entry is live, and otherwise its notice is queued through `QueueOrchRevive`.
The report says `queued` or `already-queued`, that the notice is conditional, and whether a watcher is live, from `OrchWatcherLive`, so a queued notice with no watcher reads as such.
Tender never types into the orch's pane (`PATTERN-orch-pane-single-writer`).
An orch in any other state is reported skipped with its state; with no orch strand or no `Prime`, the report has no orch part.

Bound: `revive` types one fixed line only into a session whose state reads idle-stalled with cause `api-error` at the conditioned send's own ready reading, or, for the orch, at the notice's delivery, and only through the conditioned send or the orch notice queue.
It sends no key, kills nothing and changes no live entry.
A second `revive` finds a revived session busy or done and sends nothing, and finds the orch's notice still queued and queues none.
A `pause`, `resume` or orch watcher record write waits at most one conditioned send's hold, since `pauserecord.Update` blocks on the lock with no timeout.
Coverage reads take no lock, so the send's own reads never wait on it.
A pause added while a line is in flight takes its entry only after that session's turn start reads or its turn-start wait ends.
When the turn start read within the wait, the pause's grace poll reads the session busy and its interrupt pass records it, so `resume` gives it its line.
A send that lands with no turn start seen within the wait is reported `sent_no_turn_start`, and `lyx tender status` shows the session busy under any live entry, so the operator can pause that slug again.

`reviveCmd()` is `revive [<slug>]`, mounted under `tender` by `Command`, with the operator audience.
It applies `pause`'s caller rule through `refuseCaller`: the orch and an external caller may name any scope, and an agent only its own pair, where its own busy session reads skipped and only its siblings can be revived.
It refuses a slug with no pair, naming the slugs that have one, as `pause` does, and prints the report under `revive`.
`refuseCaller`'s message names all three verbs and keeps the way forward "ask your parent session", which runs `lyx pause`, `lyx resume` or `lyx tender revive` for it.
`hubDeps` wires:
- `OrchStrand` from `orchengine.LoadState(orchcli.PrimePaths(prime)).Strand`;
- `QueueOrchRevive` as `orchengine.QueueConditionalNotice(orchcli.PrimePaths(prime), line, orchengine.NoticeCondition{State: shuttleengine.SessionIdleStalled, Cause: shuttleengine.SessionCauseAPIError}, time.Now())`.

`refusal-spec.md`'s pause section gains the row "revive: unknown slug", with the way forward the refusal carries, and its existing caller row, "pause or resume: agent outside its own pair", names revive in its title, trigger and way forward.
`docs/overview.md`'s tender bullet, `pausecli/doc.go` and `pauseengine/doc.go`, in a "Revive" section, describe the verb, its lock hold and its bound.
`SANDBOX-CORE-SUITE.md`'s S12 asks whether `lyx tender revive` with nothing stalled prints an empty report and succeeds.

`fakePair` gains `conditioned []string`, the guids sent to, `conditionedErr map[string]error`, `noTurnStart map[string]bool` and `onConditioned func(guid string)`, a hook run inside `SendConditioned` before it returns.
`SendConditioned` fails a text other than `ReviveLine`, as `Send` does for the resume line.
`env` gains `orchStrand string`, `orchQueued []string` and `orchAlreadyQueued bool`, which `env.deps` turns into `OrchStrand` and `QueueOrchRevive`.
As card 9 does for `Prime`, `env.deps` leaves `Prime` nil when `e.prime` is nil, so a row with no prime never calls a nil fake.

`TestRevive`, the one new top-level test, in `resume_test.go`, over the fake clock:
- nothing stalled: every list is empty, there is no orch part, and nothing was sent.
- states: `a` idle-stalled `api-error` is sent; `b` busy, `c` idle-done and `d` idle-stalled `interrupt` are skipped; `SendConditioned` ran for `a` only.
- covered: a live slug entry for the pair reports `a` paused with no send.
- under the lock: `onConditioned` asserts that `lock.TryAcquireWriteLock` on the record's `.lock` path fails while the send runs.
  It then starts `Pause` on a goroutine and scripts `a` busy; after `Revive` returns and the goroutine ends, the pause's entry records `a` and the fake recorded its interrupt.
- no turn start: `noTurnStart` reports `a` under `sent_no_turn_start`.
- refusals: a `*shuttleengine.SessionBusyError` reports `refused` with its reason; `ErrSubmissionNotLanded` reports `not_landed`; each pair was sent to once.
- turned busy: `ErrSendConditionUnmet` reports `a` skipped, with one send; a second `Revive` with `a` now idle-done sends nothing.
- state read failure: `stateErr` fails `a`'s `State`; `a` is reported `refused` with the error and nothing is sent.
- orch: a prime fake with the orch's session idle-stalled `api-error` and a second prime session also stalled; the notice is queued once and the orch part reads `queued`, and the second prime session is neither sent to nor reported.
  Run with `watcherLive` true the part reads the watcher live, and with `watcherLive` false it reads queued with no watcher live.
  With `orchAlreadyQueued` the part reads `already-queued`; with a hub entry live it reads `paused` and nothing is queued; a slug-scope revive has no orch part.

`TestPauseAndResume_WriteLiftAndRefuseOverTheHub` gains rows: `revive` with nothing stalled succeeds with an empty report, and `revive no-such-pair` is refused listing the pair's slug.
`TestPauseAndResume_ApplyTheCallerRules` gains rows: an agent's hub-wide `revive` and its `revive other-pair` are refused naming the parent session, and its `revive` of its own pair succeeds.

`test-budget.yaml`'s `internal/pauseengine` row rises from 4 to 5.

**ImpactSummary:** `pauseengine`'s Pair interface and Deps gain the revive seams with their two implementers adapted; pause, resume and status run unchanged, and the caller refusal keeps its way forward.

**Commit:** 10: tender revive: one carry-on line to each session stalled on an API error
