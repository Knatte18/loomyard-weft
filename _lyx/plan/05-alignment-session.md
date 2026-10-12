# Card 5 — alignment-session

**Create:**
- `internal/alignment/session_test.go`
- `plan:internal/alignment#Shuttle` -> `type Shuttle interface`
- `plan:internal/alignment#Run` -> `type Run interface`
- `plan:internal/alignment#RunnerShuttle` -> `func RunnerShuttle(r *shuttleengine.Runner) Shuttle`
- `plan:internal/alignment#SessionConfig` -> `type SessionConfig struct`
- `plan:internal/alignment#Session` -> `type Session struct`
- `plan:internal/alignment#NewSession` -> `func NewSession(cfg SessionConfig) *Session`
- `plan:internal/alignment#SpawnInput` -> `type SpawnInput struct`
- `plan:internal/alignment#Outcome` -> `type Outcome struct`
- `plan:internal/alignment#CauseTimeout` -> `const CauseTimeout = "timeout"`
- `plan:internal/alignment#CauseDied` -> `const CauseDied = "died"`
- `plan:internal/alignment#CauseMissing` -> `const CauseMissing = "missing"`
- `plan:internal/alignment#CauseInvalid` -> `const CauseInvalid = "invalid"`
- `plan:internal/alignment#CauseClosed` -> `const CauseClosed = "closed"`
- `plan:internal/alignment#Session.Begin` -> `func (s *Session) Begin(in SpawnInput) (Attempt, error)`
- `plan:internal/alignment#Session.Poll` -> `func (s *Session) Poll(a Attempt) (Outcome, error)`
- `plan:internal/alignment#Session.Close` -> `func (s *Session) Close(a Attempt, reason string) error`
- `plan:internal/alignment#Session.MarkApplied` -> `func (s *Session) MarkApplied(a Attempt, applied string, notified bool) error`
- `plan:internal/alignment#Session.Attempts` -> `func (s *Session) Attempts() ([]Attempt, error)`
- `plan:internal/alignment#Session.watchDone` -> `func (s *Session) watchDone(number int) <-chan struct{}`
- `plan:internal/alignment#fakeShuttle` -> `type fakeShuttle struct`
- `plan:internal/alignment#TestSession` -> `func TestSession(t *testing.T)`

**Edit:**
- `internal/alignment/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the agent's lifecycle in a new file `session.go`, shared by both points: spawn once per key, poll without blocking, close, and record what Go applied.
The arbiter in `internal/loomcli/arbiter.go` is the precedent; this session differs in that it never blocks the caller, since the parent-review gate polls it on each re-evaluation.

The seams:
- `Shuttle` is the slice of shuttle the session drives: `Start(spec) (Run, error)`, `Probe(spec) (Run, bool, error)` and `StopStrand(guid) error`.
- `Run` is `Wait() (shuttleengine.Result, error)` plus `StrandGUID() string`.
- `RunnerShuttle` adapts `*shuttleengine.Runner`, with `Probe` over `Runner.ProbeGated` and a zero gate.

`SessionConfig` carries `Root` (one point's attempt directory, `shedrun.AlignmentDir`), `StencilsDir`, `Shuttle`, `Now` (nil means `time.Now`) and `Token` (nil means eight random bytes as hex from `crypto/rand`).
`Session` keeps the config and an in-process map from attempt number to a watch, guarded by a mutex: a goroutine waiting on the attempt's `Run` that stores its result and then closes the watch's done channel.
`watchDone(number)` returns that channel, or a closed one when no watch exists, so a test waits on it after releasing a run and the next `Poll` reads the stored result deterministically.
`SpawnInput` carries `Key`, `Prompt` (a `PromptInput` whose verdict and report paths the session fills), `Settings`, `Snapshot`, `PRNumber` and `HeadSHA`.
`Outcome` carries `Done`, `Verdict`, `Cause` (empty for the agent's own valid verdict, else one of the `Cause*` constants) and `Detail`.

`Begin(in)`:
- If an attempt under `Root` has the same `Key`, it returns that attempt and starts nothing, closed or not, so a driver restart never spawns a second agent for one round or one pull request head (D4, D5).
- Otherwise it numbers the attempt one past the highest, draws the token, and sets `SpawnedAt` and `Deadline` (now plus `Settings.Timeout`).
- It writes `attempt.json` before the spawn, then removes any file already at the attempt's verdict or report path, so only a verdict written after this spawn can be read (D2's bound).
- It renders `Prompt`, writes it to `PromptPath`, starts `Spec(prompt, attempt, settings)`, records `StrandGUID` in `attempt.json` and starts the watch.
- A failed start closes the attempt with the start error as its reason and returns the error. A failed strand record stops the strand before returning, so nothing runs untracked.

`Poll(a)` re-reads `attempt.json` and never blocks:
- A closed attempt is `Done` with `CauseClosed`, and its verdict file is never read, so a verdict written after a fallback, a parent verdict or a stop is never applied.
- A verdict file present is read through `ReadVerdict`: valid gives `Done` with the verdict, invalid gives `Done` with `CauseInvalid` and the reason.
- With a watch whose run ended without a verdict: `CauseTimeout` for a shuttle timeout, `CauseDied` for a died run or a wait error, and `CauseMissing` for a run done with no verdict. A live strand is stopped through `StopStrand` first.
- With no verdict once `Now` reaches `Deadline`, it stops the strand and answers `CauseTimeout`.
- With no watch in this process, after a driver restart, it probes `Spec` with the remaining time. A live run is watched and reads as pending; no live run gives `Done` with `CauseDied`, "no agent is live for the attempt and it left no verdict".
- Anything else is pending.

`Close(a, reason)` stops a strand that may still be live through `StopStrand`, returning a stop error since an agent must not run beside a fallback, and then marks the attempt closed with the reason.
Closing an attempt that is already closed changes nothing.
`MarkApplied(a, applied, notified)` writes `Applied` and `Notified` into `attempt.json`, and `Attempts()` is `ListAttempts(Root)`.
A stop or close is logged through `internal/logger` (`PATTERN-spawn-observability`).

`TestSession` is one table test over `fakeShuttle`, which records starts, probes and stops and hands out runs whose `Wait` blocks until the row releases a result, with a fixed token and an injected clock. Rows:
- A first `Begin` writes `attempt.json` and `prompt.md` naming the verdict and report paths, and starts one spec whose `OutputFiles` are those two paths and whose role is `alignment`. `Poll` is pending until the row writes both files, then `Done` with the verdict.
- A second `Begin` with the same key starts nothing and returns the same attempt, closed or open.
- A file planted at the next attempt's verdict path before `Begin` is gone after it, and `Poll` is pending.
- Reaching the deadline stops the strand and gives `CauseTimeout`. A run released as died gives `CauseDied`, and one released done with no files gives `CauseMissing`; each of these rows waits on `watchDone` before it polls.
- An invalid verdict, an `escalate` with no question, gives `CauseInvalid`.
- `Close` on a live attempt stops its strand once, and a verdict written afterwards polls as `CauseClosed`.
- A fresh `Session` over the same root, standing in for a restart, probes once: a found run is pending, and no run gives `CauseDied`.
- `MarkApplied` round-trips through `Attempts`.

`doc.go` describes the lifecycle, and `test-budget.yaml` raises `internal/alignment` to 7.

**ImpactSummary:** Adds the session type to the card-4 package; no existing package calls it until cards 7 and 9.
