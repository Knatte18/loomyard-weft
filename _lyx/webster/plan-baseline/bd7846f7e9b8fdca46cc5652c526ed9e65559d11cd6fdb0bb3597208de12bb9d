# Card 3 — lost-answer-names-done

**Edit:**
- `internal/creelcli#result`
- `internal/creelcli#refuse`
- `internal/creelcli#writeRefusal` -> `func writeRefusal(cmd *cobra.Command, done, msg string, fields map[string]any, code int)`
- `internal/creelcli#runVerb`
- `internal/creelcli#postCmd`
- `internal/creelcli#injectCmd`
- `internal/creelcli#interruptCmd`
- `internal/creelcli#forwardCmd`
- `internal/creelcli#holdCmd`
- `internal/creelcli#releaseCmd`
- `internal/creelcli/doc.go`
- `internal/creelcli#TestCreelCLI`

**Create:**
- `plan:internal/creelcli#partialWriter` -> `type partialWriter struct`
- `plan:internal/creelcli#partialWriter.Write` -> `func (w *partialWriter) Write(p []byte) (int, error)`
- `plan:internal/creelcli#partialWriter.String` -> `func (w *partialWriter) String() string`

**Uses:**
- `internal/creelcli#writeOk`
- `internal/creelcli#runCreel`
- `internal/creelcli#runCreelSplit`
- `internal/creelcli#runCreelInto`
- `internal/creelcli#failingWriter`
- `internal/creelcli#lockedBuffer`
- `internal/creelcli#newRig`
- `internal/creelcli#rig.startDaemon`
- `internal/creelcli#rig.name`
- `internal/creelcli#rig.awaitLogged`
- `internal/creelcli#rig.sweepStep`
- `internal/creelcli#rig.sweepLast`
- `internal/creelcli#isProbeWait`
- `internal/creelcli#isSweepWait`
- `internal/testkit/creelfake#LiveStrand`
- `internal/testkit/creelfake#Delivery.SetStrands`
- `internal/testkit/creelfake#Delivery.SetIdle`
- `internal/testkit/creelfake#Delivery.SetInterruptError`
- `internal/testkit/creelfake#Delivery.SetSendError`
- `internal/testkit/creelfake#Delivery.SetStall`
- `internal/testkit/creelfake#Clock.AwaitWaitsWhere`
- `internal/testkit/creelfake#Clock.AdvanceByWaitsWhere`
- `internal/creelcli#awaitStderr`
- `internal/creelengine#ErrNoRunningTurn`
- `internal/creelengine#StallDraft`
- `internal/agentname#StrandNameEnv`

**Intent:**
D2: `result` gains a field `done string`, the verb's own statement of what it already did, set by the bodies that store or change something before the answer is written:
- `postCmd`, both a post to a receiver (with or without `--wait-reply`, including the returned-undelivered result) and `--reply`: `message #<resp.ID> was posted`;
- `injectCmd`, every result built from a response (plain, injected, unconfirmed, expired): `inject #<resp.ID> was queued`;
- `forwardCmd`, `holdCmd`, `releaseCmd`, and `interruptCmd` when `resp.Sent` is true: `the <verb> took effect` (`the forward took effect`, `the hold took effect`, `the release took effect`, `the interrupt took effect`).
An interrupt that found no running turn, `inbox`, `log` and `read` leave `done` empty.
A result returned beside an error, the wait timeout included, carries no `done`, since the client answers those with an empty response and the body holds no id.

`writeRefusal` takes `done` before `msg`.
Its stderr fallback writes `creel: <done>; <msg>` when `done` is non-empty, and otherwise the line card 2 left (`creel: ` prepended unless `msg` opens with it); the JSON envelope is unchanged and carries no `done`.
`refuse` passes an empty `done`; `runVerb` passes `""` on the wait-timeout path and `res.done` on the non-zero-code path, so the unconfirmed, expired and returned fallbacks name the id.

D1: `runVerb`'s zero-code path writes `res.fields` with `writeOk(out, res.fields)` instead of `output.Ok`.
When that write fails it records exit 1 with `clihelp.SetExit` and writes one line straight to `cmd.ErrOrStderr()`, never through `writeRefusal`, so no error envelope follows on stdout:
`creel: <done>, but its answer could not be written: <write error>` when `res.done` is set, else `creel: the answer could not be written: <write error>`.
The stderr write's own error is ignored, as `writeRefusal` does, so with both streams refusing the exit is still 1.
This covers every verb answering through `runVerb`'s zero-code path: post, inject, interrupt, forward, inbox, log, hold and release; `read` keeps its own `printed` path and the hidden `daemon` verb keeps `output.Ok`, so the `output` import stays.
Bound: only the success envelope's write is newly checked; the daemon's stored message, inject, hold or interrupt is not undone, and a write that fails after part of the line went out still exits 1 with the stderr line.
Update the doc comments of `result`, `writeRefusal` and `runVerb` to state the field and the new line.

D5: `internal/creelcli/doc.go` gains: a success envelope that cannot be written exits 1 with one `creel: ` answer line on stderr, after any stall notes, naming what post and inject already stored or saying that a forward, hold, release or interrupt took effect; and the refusal fallback line opens with that statement too for a post or inject that stored before it refused.

Tests in `TestCreelCLI`; each stderr check below means the stderr buffer holds exactly one line, unless the row says stall notes precede it, compared exactly or against an anchored `regexp` (add the `regexp` import), and an id is read from the line's `#(\d+)`.

A new test helper, `partialWriter`, is a stdout that takes the first half of its first write and then refuses it, and accepts and records every later write; it guards its buffer with a mutex like `lockedBuffer` and has a `String` reader the rows use.

Extend the subtest `a waiting verb's envelope that cannot be written keeps its exit code and goes to stderr`:
- the rig's strands are `alpha` and `beta` (`r.delivery.SetStrands(creelfake.LiveStrand(alpha), creelfake.LiveStrand(beta))`);
- first, with `r.delivery.SetSendError(creelengine.ErrNotLanded)` set, a written unconfirmed row: `runCreelSplit(r.cwd, &stderr, "inject", "--wait", alpha, "/clear")` exits `ExitInjectUnconfirmed`, its stderr is empty, its envelope's `error` is `the text was typed but its landing was not confirmed`, and the envelope has no `done` key;
- the unconfirmed row's line now matches `^creel: inject #(\d+) was queued; the text was typed but its landing was not confirmed$`, and `runCreel(r.cwd, "log", "--id", <id>)` lists a `posted` event, so the inject is stored;
- a new expired row, with stall notes, placed after the unconfirmed row: `r.delivery.SetSendError(nil)` and `r.delivery.SetStall(alpha, creelengine.StallDraft)`; `probesBefore := len(r.clock.AwaitWaitsWhere(0, isProbeWait))`; run `inject --wait --ttl 1m alpha /clear` into `failingWriter{}` in a goroutine that closes a `returned` channel when it ends; `r.clock.AwaitWaitsWhere(probesBefore+1, isProbeWait)`, `r.clock.Advance(time.Second)`, `awaitStderr(t, &stderr, "is stalled: "+creelengine.StallDraft.Reason())`, then `r.clock.AdvanceByWaitsWhere(isProbeWait, returned)`, as the subtest `a waiting inject hears the stall on stderr and an expired answer carries it` drives it.
  It exits `ExitInjectExpired`; every stderr line but the last holds `is stalled: `, and the last line matches `^creel: inject #(\d+) was queued; the inject expired before it could be typed$`.
  Then `r.delivery.SetStall(alpha, "")`;
- the timeout row, after `r.delivery.SetIdle(alpha, false)`, keeps exit `ExitWaitTimeout` and its line is exactly `creel: no outcome arrived before the timeout of 5s`, naming no id;
- a new returned row: run `post --wait-reply beta anyone?` into `failingWriter{}` in a goroutine, `r.awaitLogged(t, "anyone?")`, `r.clock.AwaitWaitsWhere(1, isSweepWait)`, `r.delivery.SetStrands(creelfake.LiveStrand(alpha))`, `r.sweepStep()`, `r.sweepLast()`; it exits `ExitReturnedUndelivered` and its line matches `^creel: message #(\d+) was posted; the message returned undelivered$`.

Add one subtest, `a verb whose answer cannot be written exits 1 with one creel: line naming what it did`, on a rig with strands `alpha` and `beta` and a running daemon, each row run through `runCreelInto(r.cwd, failingWriter{}, &stderr, ...)` with its own `lockedBuffer`:
- written answer: `runCreelSplit(r.cwd, &stderr, "post", alpha, "seen")` exits 0 and its stderr holds no `creel: ` line;
- as EXTERNAL, `post alpha hello` exits 1 with a line matching `^creel: message #(\d+) was posted, but its answer could not be written: write refused$`, and `log --id <id>` lists `posted`;
- `inject alpha /clear` exits 1 with `^creel: inject #(\d+) was queued, but its answer could not be written: write refused$`, and `log --id <id>` lists `posted`;
- `interrupt alpha` (the fake's interrupt succeeds) exits 1 with `creel: the interrupt took effect, but its answer could not be written: write refused`;
- after `r.delivery.SetInterruptError(creelengine.ErrNoRunningTurn)`, `interrupt alpha` exits 1 with `creel: the answer could not be written: write refused`; then `SetInterruptError(nil)`;
- `hold alpha --for 1m` exits 1 with `creel: the hold took effect, but its answer could not be written: write refused`, and `log --to alpha` holds a `held` event, so the hold stands;
- `release alpha` exits 1 with `creel: the release took effect, but its answer could not be written: write refused`;
- `log` exits 1 with `creel: the answer could not be written: write refused`;
- `log` with `failingWriter{}` as both stdout and stderr exits 1;
- `log` into a `partialWriter` exits 1 with `creel: the answer could not be written: ` and the write's error, and the writer holds only the half line it took, with no `"ok":false` envelope after it, so the lost answer never goes through `writeRefusal`;
- then as the strand `alpha` (`t.Setenv(agentname.StrandNameEnv, alpha)`, its `LiveStrand` given `PanePID = os.Getpid()` through `SetStrands`, beside `beta`): `inbox` exits 1 with `creel: the answer could not be written: write refused`;
- `post alpha "note to self"` and `read` run written; `post --reply <that id> answer` exits 1 with `^creel: message #(\d+) was posted, but its answer could not be written: write refused$` naming an id other than the replied one, and `log --id <that id>` lists `posted`, so the reply is stored;
- `post alpha "pass on"` runs written, and `forward <its id> beta` exits 1 with `creel: the forward took effect, but its answer could not be written: write refused`.

Old texts grepped: no test asserts `output.Ok`'s output on these paths, and the existing failing-writer assertions (`creel: ` prefix, `not confirmed`, `no outcome arrived`) hold under the new lines.

**ImpactSummary:** `writeRefusal`'s two callers, `refuse` and `runVerb`, are on this card; the bodies of six verbs set one new field, and only the stderr path of a failed success write and of a stored-then-refused post or inject changes, all inside `internal/creelcli`.

**Commit:** 3: creel names what a verb did when its answer cannot be written
