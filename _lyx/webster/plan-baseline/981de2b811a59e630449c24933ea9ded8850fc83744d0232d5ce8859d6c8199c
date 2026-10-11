# Card 2 — pattern-headroom

**Create:**
- `plan:internal/pattern#HeadroomBytes` -> `const HeadroomBytes = MaxOverviewBytes * 95 / 100`
- `plan:internal/pattern#KindOverviewHeadroom` -> `const KindOverviewHeadroom = "overview-headroom"`
- `plan:internal/pattern#Headroom` -> `func Headroom(base, head []byte) (Finding, bool)`
- `plan:internal/pattern#TestHeadroom` -> `func TestHeadroom(t *testing.T)`

**Edit:**
- `internal/pattern#Check`
- `internal/pattern#TestCheck`
- `internal/pattern/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Give `internal/pattern` the headroom line and the pure growth check of decisions D1, D2 and D5, keeping the package a leaf: nothing here reads git or disk, and its import set stays closed.
`HeadroomBytes` is derived from `MaxOverviewBytes` by integer arithmetic, never a second literal, and sits beside it in `check.go`.
`Headroom` lives in a new `internal/pattern/headroom.go` with its test in `internal/pattern/headroom_test.go`.
It takes the overview's content at a base and at the head, where a nil or empty base means the overview was absent at the base and counts as zero bytes, and returns a `Finding` of kind `KindOverviewHeadroom`, subject `PATTERN.md`, and true exactly when the head is longer than the base and longer than `HeadroomBytes`.
An absent head, nil or empty, passes.
The finding's message names the head size, the base size, the line and the bytes to cut, which is the head size minus the larger of the base size and the line: the smallest cut that passes, so a fixer whose base already sits above the line is told to cut only the branch's own growth.
`TestHeadroom` is one table-driven test whose rows are the acceptance criteria: grew from under the line to above it fails and names the cut to the line; grew from a base above the line fails and names the cut back to the base size; grew to exactly the line passes; shrank while above the line passes; unchanged above the line passes; absent at the base and present above the line fails naming the cut to the line; absent at the head passes.
Each failing row asserts the exact message.
`Check`'s oversize-overview message gains the number of bytes over the cap beside the size and the cap, and `TestCheck`'s oversize row asserts that message, through a new optional `wantMessage` field on its row struct.
`internal/pattern/doc.go`'s format section states the headroom rule: the hard cap `Check` enforces on any tree, the headroom line `Headroom` binds growth to, the cut it names, and that the line binds a branch's own change while a merged tree answers only to the cap.
The `internal/pattern` row of `cmd/lyx/testdata/test-budget.yaml` rises by one.

**ImpactSummary:** Adds three exported symbols to the pattern leaf; Check's oversize message gains the bytes over the cap, read by `TestCheck`, by `TestOverview_RealFilePassesCheck`'s failure output and through it by Publish's failure record, and no code parses the message.
