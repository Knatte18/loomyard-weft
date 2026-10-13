# Card 2 — github-held-deadline

**Edit:**
- `internal/githubclient#authRT`
- `internal/githubclient#authRT.RoundTrip`
- `internal/githubclient#NewWithBaseURL`
- `internal/githubclient#clientTimeout`
- `internal/githubclient/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/githubclient#authRT.requestContext` -> `func (rt *authRT) requestContext(caller context.Context, held time.Duration) (context.Context, context.CancelFunc, error)`
- `plan:internal/githubclient#cancelOnClose` -> `type cancelOnClose struct`
- `plan:internal/githubclient#TestAuthRT_HeldCallGetsItsFullDeadlineAfterRelease` -> `func TestAuthRT_HeldCallGetsItsFullDeadlineAfterRelease(t *testing.T)`

**Intent:**
A GitHub call a pause holds gets its full deadline after the hold ends, so the hold no longer uses up the call's budget (D6, issue #606).

`NewWithBaseURL` drops the `http.Client`'s own `Timeout` and builds the transport as `&authRT{transport: inner, timeout: clientTimeout}`.
The request context then carries only the caller's deadline.
`authRT` gains a `timeout time.Duration` field, the client timeout it applies itself; zero means none, which keeps the transport tests that build `&authRT{}` unbounded.
`clientTimeout`'s comment says it starts after the pause hold and bounds the original attempt and a 401 replay together.

`authRT.RoundTrip` runs `held, done := pauserecord.Network("github")`, then derives the context the request goes out under through `requestContext`, and sends a clone of the request with that context, through the 401 replay as today.
`requestContext(caller, held)`:
- returns the caller's error at once, sending nothing, when the caller's context reads `context.Canceled` at release;
  a call whose context was cancelled during the hold, before its own deadline passed, so ends with the cancellation once the hold releases.
- for `held` zero, uses the caller's context unchanged, so an unheld call keeps exactly its deadlines;
  a call the gate admitted before a pause started is unheld, so it is neither held nor extended.
- for `held` above zero, uses `context.WithoutCancel(caller)`, keeping the caller's values, under the caller's deadline moved later by `held` when the caller has one.
  A `context.AfterFunc` on the caller cancels it when the caller ends with `context.Canceled`, so a cancellation that arrives before the caller's own deadline passes still ends the call;
  once that deadline has passed, the caller's error reads `context.DeadlineExceeded` and a later cancellation is not observable, so only the moved deadline ends the call.
  The extension is therefore the longest a late cancellation can be ignored.
- then applies `rt.timeout`, when positive, as a timeout from now, the end of the hold.

The derived context must outlive `RoundTrip` while the caller reads the body: on a transport error the cancel runs at once, and on a response the body is wrapped in `cancelOnClose`, whose `Close` closes the body and then cancels.
A caller issuing several requests under one deadline gets the extension for the held request only, since each request measures its own `held`.
The gate stays blind to the caller's context, and `pauserecord.Network` keeps its context-free signature;
the extension logic lives here, and `pauserecord` keeps its import set.

Bound: the extension equals the time the gate held that one request, so it never lets an unheld call run longer than today, and a held call runs at most its own budget after the hold.

`doc.go`'s hold paragraph and its "The two deadlines" section say that the 30 s timeout is applied by `authRT` from the end of the hold, no longer by `http.Client.Timeout`, and that a held call's caller deadline moves later by the held time.

`TestAuthRT_HeldCallGetsItsFullDeadlineAfterRelease` is a new top-level test in `githubclient_test.go`, not parallel since the network gate is process-wide state, as for `TestAuthRT_WaitsWhileAPauseCoversTheProcess`.
It covers the transport's deadlines, which no existing test reaches.
Its rows run a request through `&http.Client{Transport: &authRT{timeout: 50 * time.Millisecond}}` with a caller deadline of 50 ms, against `newScriptedServer` or a handler of the row's own:
- a held call: a gate over a live hub entry whose `Sleep` sleeps 150 ms and then lifts the entry; the server answers at once, and the call succeeds.
  Without the fix the caller's deadline and the client timeout have both passed at release, and the call fails with `context.DeadlineExceeded`.
- an unheld call: no live entry, and a handler that answers after 150 ms; the call fails with `context.DeadlineExceeded` and the gate's `Sleep` never ran.
- a call admitted before the pause: the handler writes a live hub entry, then answers after 150 ms; the call fails with `context.DeadlineExceeded` and the gate's `Sleep` never ran.
- cancelled during the hold: the gate's `Sleep` cancels the caller's context, with no deadline passed, and lifts the entry; the call ends with `context.Canceled` and the server received nothing.
- cancelled after release: a caller context with a 10 s deadline and a handler that blocks until the test cancels that context once the request arrives; the call ends with `context.Canceled` well within the deadline.
- cancelled after its own deadline passed during the hold: a 50 ms caller deadline and a gate `Sleep` of 150 ms, so the moved deadline is about 200 ms.
  The handler blocks, and the test cancels the caller once the request arrives.
  The call keeps running past the cancel and ends with `context.DeadlineExceeded` at the moved deadline.
  An implementation that cancels on any end of the caller's context fails this row.

With the hold no longer using up the budget, a GitHub deadline error means the call's own budget ran out after release.
It stays classified transient, and the loom drive re-steps it once before the tackler judges it, so a first occurrence never halts the run `failed` (#606).
That chain is pinned by existing tests and gets no new row:
- `internal/githubclient#TestIsTransient`'s row "wrapped url deadline" pins the deadline classification;
- `internal/shedtransient#TestClassAndMark`'s "github 503" row pins `Class` mapping `IsTransient` onto `TransientGitHubAPI`;
- `internal/landingshed#TestPRGate_ReturnedErrors`'s "transient query failure is classified" row pins a landing producer returning such an error classified;
- `internal/shedverbs#TestLoop_TransientRestepOnce` pins the once-only re-step.
`loomengine.ClassifyHalt` keeps labelling a spent re-step `producer-hard-failure`, unchanged.

`test-budget.yaml`'s `internal/githubclient` row rises from 12 to 13.

**ImpactSummary:** The transport and client constructor of the one GitHub client package every consumer uses; unheld calls keep their deadlines bit for bit, and only a held call's deadline changes.

**Commit:** 2: githubclient: a held call gets its full deadline after release
