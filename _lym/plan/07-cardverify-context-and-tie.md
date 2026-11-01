# Card 7 — cardverify-context-and-tie

**Edit:**
- `internal/cardverify#Runner`
- `internal/cardverify#Runner.RunLine` -> `func (r Runner) RunLine(ctx context.Context, line string, out io.Writer) string`
- `internal/cardverify#Runner.RunCards` -> `func (r Runner) RunCards(ctx context.Context, cards []planparser.Card, out io.Writer, stopAtFirst bool) []string`
- `internal/cardverify#Runner.runLine` -> `func (r Runner) runLine(ctx context.Context, line, site string, out io.Writer) string`
- `internal/cardverify#Runner.acquire` -> `func (r Runner) acquire(ctx context.Context, site string) (*gateslot.Lease, error)`
- `internal/cardverify#Runner.execute`
- `internal/cardverify#TestRunner`
- `//internal/cardverify/doc.go`
- `internal/gatecli#runCardVerify`
- `internal/websterengine#rerunCardVerifies`

**Uses:**
- `internal/verifyrun#RunTied`

**Intent:** D6: `cardverify.Runner` takes the caller's context in place of its own `context.Background()`, for the slot acquire and the line, so a lease cancel reaches both, and it can tell its caller where it stands and which process tree runs the line.

`RunLine`, `RunCards`, `runLine` and `acquire` take `ctx`; `runLine` passes it to `execute` (the bound, when set, is derived from it) and to `acquire`, whose `Slots.Acquire` waits under it, so a cancel during the slot wait returns `could not acquire a gate slot: context canceled` and a cancel during the line ends its process group through `verifyrun`'s group terminate, SIGTERM then SIGKILL.
`Runner` gains two optional fields, nil meaning today's behavior:
`Phase func(slotWait bool)`, called with true right before a slot-held line's acquire and with false right before any line runs, so a caller can name the slot wait and the run apart;
`Tied func(proc.TreeRecord)`, which makes `execute` run the line through `verifyrun.RunTied` with it as `started`, so the caller can record the line's process group.
The callers pass the context they have: `runCardVerify` passes its `ctx`, and `rerunCardVerifies` passes `context.Background()`, since record-batch's re-run has no cancel of its own.
The cardverify doc states the context, the two fields and that a cancelled line ends its process group.

`TestRunner`, tier1, passes `context.Background()` where it calls the methods and gains parallel subtests:
- a context cancelled while a slot-held line waits for a pool whose one slot the test holds returns a `could not acquire a gate slot` failure and leaves the slot to its holder;
- a context cancelled while the line `sleep 30 & echo $! > <file>; wait` runs returns within the wait delay, and the backgrounded sleep is gone;
- `Phase` sees `true, false` for a slot-held line and `false` alone for an unslotted one;
- `Tied` receives a record whose PID is the line's shell, written by the line itself as `$$` into a file.

**ImpactSummary:** four cardverify methods take a context and the Runner gains two optional fields; the two production callers pass their context, and nothing else calls them.
