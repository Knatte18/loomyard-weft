# Card 5 — The parent-review gate closure

**Create:**
- `internal/parentreview/gate.go`
- `internal/parentreview/gate_test.go`
- `plan:internal/parentreview#GateConfig` -> `type GateConfig struct`
- `plan:internal/parentreview#NewGate` -> `func NewGate(cfg GateConfig) (gate, final shuttleengine.Gate)`

**Edit:**
- `internal/parentreview/seam_enforcement_test.go`

**Uses:**
- `plan:internal/parentreview#Store`
- `plan:internal/parentreview#Round`
- `plan:internal/parentreview#Store.Latest`
- `internal/shuttleengine#GateResult`
- `internal/shuttleengine#Gate`

**Intent:**
Implement decision D4's closure and D5's prompt schedule over the store, as the two closures card 1's `GateEntry.Gate`/`GateEntry.Final` take.

`GateConfig` carries the `Store`, the run slug, the reviewer full name from the seed (empty when none), the decision record and support log paths, `WaitBound time.Duration`, and two render seams: `RenderDelivery func(requestPath string) (string, error)` and `RenderBrief func() (string, error)`.
The seams keep stencil reading and every provider word outside this package and outside `shuttleengine`.

`gate`, evaluated at each arrival and turn-boundary tick, reads the latest round and returns:

- no reviewer: passed, with one Info line per closure;
- no request in the latest round (or no round at all): opens it (`request.json`, `brief.md` from `RenderBrief`), records one carried prompt, and returns pending carrying `RenderDelivery`'s text;
- open, no verdict, `now - opened-at >= WaitBound`: marks it expired and returns passed with a Warn naming the reviewer and the request path;
- open, no verdict, inside the bound: when a notify is waiting, takes one and carries the prompt outside the cap and throttle; else, with no delivered-at, fewer than three gate prompts and a minute since the last prompt, records one more and carries the prompt; else, once three are spent, logs one Warn per round that re-sending stopped; returns pending, with `SendFailedWayForward` naming `lyx loom review notify <slug>`;
- verdict approve: passed;
- verdict reject, unconsumed: marks it consumed and returns failed with findings naming the copied `review.md` and telling the writer to address every finding by fix or answer, as its prompt's parent-review section describes;
- verdict reject, consumed: passed;
- an expired or superseded latest round: passed.

`final`, evaluated by `finalize`, never opens a request, carries no prompt and consumes nothing: an open request without a verdict is marked expired and returns pending (reported `waiting`); an unconsumed reject logs a Warn naming its review file and returns passed; every other state returns passed.
The wait bound is measured from opened-at, so a restart never resets it, and every count lives on disk, so the three-prompt cap and the minute throttle hold across driver restarts.

Bound: the prompt reaches only the writer's own session, at most three gate prompts per round plus one per notify, and the closure never sends anything itself.

`gate_test.go` drives the state table with a fake clock and fake render seams: open-and-prompt, the throttle, the cap and its single Warn, notify past the cap, `--failed` not stopping re-sends, expiry from the original opened-at after a simulated restart, the cap and throttle across a restart (two prompts carried, then a second `NewGate` over the same store carries one more only a minute after the second and none after the third), a notify added between two closures carried by the second closure's first call, approve, reject consumed once then passed, a consumed reject on attach, no reviewer, and `final`'s non-opening and non-consuming behaviour.
The allowlist gains `internal/shuttleengine`.

**ImpactSummary:** New closure file in the package card 4 created; the only edit widens that package's own import allowlist by one entry.
