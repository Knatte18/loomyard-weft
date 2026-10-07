# Card 6 — The strand remover D1 and D6 stop a half with

**Create:**
- `internal/burlerengine#StrandRemover`
- `internal/burlerengine#NewReedStrandRemover`
- `internal/burlerengine/remover.go`
- `internal/burlerengine/remover_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#removeStrandIfLive`
- `internal/shuttleengine#ReedOps`

**Intent:**
Add the told strand remover D1 gives `burlerengine`, as its own card because it is public surface reused across packages: the engine stops a half through it (card 7), the two CLI wirings build one, and `BurlerProducer` is told the same one for the resume path (card 8).

`StrandRemover` is `RemoveStrandIfLive(guid string) error`, the told-seam shape of `websterengine`'s `removeStrandIfLive`.
`NewReedStrandRemover` returns the reed-backed implementation over `shuttleengine.ReedOps`, with a private implementing type in `remover.go`:
it probes reed's strand table for `guid`, is a no-op for a strand reed does not report live, returns an error for a failed liveness probe (a probe that could not answer has not shown the strand dead), logs a Warn for every live strand it removes because that kills a real agent process (`PATTERN-spawn-observability`), and returns an error for a failed removal.
Nothing consumes it in this card, so the tree builds and today's round is unchanged.

What it lets through, and its bound: it kills any live strand it is handed by guid, so a caller naming the wrong guid stops an unrelated agent;
callers pass only the guid of a half's own run handle or probed run, it removes nothing for a strand reed does not report live, and a probe or removal failure is returned rather than swallowed, so the caller never proceeds beside a strand it could not stop.

`remover_test.go` is one table-driven test over a `shuttlefake.Reed`: a live strand is removed (`RemovedGUIDs` holds its guid) with a nil error;
a not-live strand and an absent guid remove nothing and return nil;
a failing `Status` returns an error and removes nothing;
a failing `RemoveStrand` for a live strand returns an error.
The table is one new top-level function, so `cmd/lyx/testdata/test-budget.yaml` raises the `internal/burlerengine` row by one (`TestTestBudget_PackagesStayWithinBudget` fails on a package over its row).

**ImpactSummary:** one added budget row; nothing else reads it, and no production code consumes the remover until card 7.
