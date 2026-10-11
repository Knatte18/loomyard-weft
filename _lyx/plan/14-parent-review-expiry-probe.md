# Card 14 — parent-review-expiry-probe

**Create:**
- `internal/parentreview#NewExpiryProbe`
- `internal/parentreview#Store.peekLatest`
- `internal/parentreview#TestExpiryProbe`

**Edit:**
- `internal/parentreview/doc.go`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#resolveGateEntry`
- `internal/shedrecipe#resolveParentReviewClosure` -> `func resolveParentReviewClosure(entry string, attempts int, env Env) (gate, final shuttleengine.Gate, expired func() (bool, error), err error)`
- `internal/shedrecipe#TestResolveGateSpec_ParentReviewResolvesMustPassMayHold`
- `internal/shedrecipe#TestResolveGateSpec_OtherNamesCarryNoFinal`

**Prosa:**
- `contracts/specs/shed-recipe-spec.md`

**Intent:**
D2, parent-review side: the probe is "the latest round's request is open, has no verdict and is older than the wait bound".
Only the parent-review closures mark a request expired, at a boundary in `closures.gate` or in `closures.final`; the probe marks nothing.

`Store.peekLatest` returns the highest-numbered round like `Latest`, reading `request.json` and `verdict.json` with `readHeldJSON`, so it takes no round lock and creates no directory; `Final` re-reads the files under the lock.
`NewExpiryProbe` returns a closure over `cfg` that answers false with no reviewer, no round, no request, a request not open or a recorded verdict, and true once the store's clock is at least `WaitBound` past the request's opened-at.
It sends, counts and writes nothing.
`doc.go` describes the probe beside the gate closures.

A verdict recorded between the probe and finalize is read by `closures.final`: an approve passes and a reject takes its terminal path, as `TestFinal_*` already pin; an open request with no verdict is marked expired there with the existing timeout Warn.

`resolveParentReviewClosure` also returns `parentreview.NewExpiryProbe` over the same config, and `resolveGateEntry` sets it as the `parent-review` entry's `Expired`.
`resolveGateSpec`'s doc and `shed-recipe-spec.md`'s `parent-review` paragraph say the entry carries an expiry probe the wait loop calls off the boundary.

Tests:
`TestExpiryProbe` is a table over `newStore`: no reviewer, no round, an open request before the bound, an approve or a reject past the bound, an expired request and a superseded one each answer false; an open request with no verdict at the bound answers true.
Every row asserts the round's files are unchanged after the call and `delivery.json` absent.
`TestResolveGateSpec_ParentReviewResolvesMustPassMayHold` asserts the entry's `Expired` is set, and `TestResolveGateSpec_OtherNamesCarryNoFinal` that every other name's is nil.

**ImpactSummary:** a read-only probe in parentreview and one more return from the shared gate resolver's parent-review branch; the gate and final closures are unchanged.
