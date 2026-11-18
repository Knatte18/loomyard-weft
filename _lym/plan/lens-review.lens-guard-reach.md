```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal, branch and test row the cards add can be reached from a real entry point.

Five findings: two BLOCKING, two MEDIUM, one LOW.

1. **BLOCKING, D3 and D9 (cards 6 and 14): the prime branch of the reach step can never be reached.**
   - Card 6 has the step write a judged-nothing report for "a worktree with no parent branch" before it computes any change set.
   - The prime has no parent branch. `parentMergeBase` returns false for "a worktree with no origin record or no parent branch, such as the prime on main" (`internal/gatecli/scope.go:690-697`).
   - So a prime call always stops at the no-parent note. Card 6's branch "the call runs in the prime → answered from the shared maps, with an `Unrefreshed` line" never runs.
   - Card 6's planned `TestGateTestReachStep` row "every package of a prime call are answered from the existing maps with their lines" cannot pass as written.
   - D3 ("a call in the prime refreshes nothing and answers from the shared maps") and its acceptance criterion go unmet.
   - Card 14 has the same problem. `lym gate mutate` in the prime is refused without `--base`, and the row "a prime row judges a package whose shared map matches the worktree" does not say it passes `--base`.
   - Fix: the prime needs a base of its own (HEAD, or a named one), or the no-parent note must come after the prime check.

2. **BLOCKING, the card 4 guard against cards 6 and 14: the guard trips on files those later cards add.**
   - From card 4 on, `TestCoverageMapReaderBoundary` fails on any non-test file outside `internal/testcover` and `internal/testreach` that calls `testcover.ReadMap`. Only `redundancy.go`, `redundant.go` and `testmappackage.go` are pinned.
   - Card 6's step "reads its worktree anchor map" so it can call `testreach.Fresh(module, pkg, m testcover.Map)`.
   - Card 14 needs a "fresh map's covered blocks" and checks the anchor or shared map with `Fresh`.
   - `internal/testreach` exposes no single-map read: `Open`, `Maps()` and `MapSource` return no `testcover.Map`. So the new gatecli files of cards 6 and 14 must call `testcover.ReadMap` directly.
   - That trips the guard, and neither card lists `internal/cmd/lym#TestCoverageMapReaderBoundary` (`cmd/lym`) to pin the new file.
   - Fix: add a testreach read of one package's map (or a fresh check by package), or have cards 6 and 14 extend the guard's pin list.

3. **MEDIUM, D3's attempt budget (card 9): the reach turn's count runs ahead of the driver's count.**
   - The order is: `WrapRowGate` runs the decorated gate, and on a failure it spawns the fixer and re-runs the gate through `RerunMark.Run`.
   - Card 9 counts the first, failing inner call (it is not a re-run). If the re-run then passes with a new unreached function, the turn "adds one to the count" again.
   - One shuttle arrival therefore counts 2, while the driver's `gateFails` counts 1 (`internal/shuttleengine/run.go:448-451`).
   - The "count plus one below `Attempts`" check then skips the turn one arrival early. That breaks D3's "the count … is one per shuttle arrival".
   - No `TestWithReachTurn` row covers a reach turn produced inside `RerunMark.Run`.
   - Fix: a turn inside a re-run adds nothing to the count, or the failure count is taken once per outer call; plus a row covering it.

4. **MEDIUM, D3 test coverage (card 9): the Darn-Write fixer path of the turn is never driven end to end.**
   - Card 8 tests the bypass with a scripted gate, and card 9 tests the decorator with a scripted inner gate.
   - Nothing composes `WrapRowGate` over `WithReachTurn` with a shared `RerunMark`, the shape `resolveGateEntry` plus `gateFixerSeam` (`internal/loomcli/gatefixer.go:225`) build in production.
   - So the criterion "with `darn.gate_fixer` set, a reach turn's result at Darn-Write spawns no cold fixer and reaches the writer" has no row that reaches it through the real wrap order.
   - Finding 3 would show up in such a row.

5. **LOW, D7 (card 11): the "lookup that fails for the test" branch cannot occur.**
   - `Reader.FuncsReachedBy(pkg, test, among)` returns `([]FuncKey, MapSource, bool)`, with no error.
   - The only failure is building the lookup (`Open`), which card 11 already handles as "adds no triage line anywhere".
   - So the planned `TestTriageLookup_Lines` row "a lookup whose reader fails gives no line" has no per-test state to set up.
   - Fix: drop the per-test branch, or give the reader an error result that can reach it.

Further calls:
- Read `internal/gatecli/scope.go:680-710` (`parentMergeBase`) and grepped `ReadOrigin`: to confirm what the prime gets. My context had only the subagent's summary of it.
- Read `internal/weavercli/wiring.go` and `internal/loomcli/wiring.go:750-765`: to check that both `WeaverGeometry` builders reach the batch gate. The weavercli one only builds reset deps, so this is not a finding.
- Grepped and read the recipe rows that carry `name: verify` (tapestry and sampler): to see which rows the `resolveGateEntry` wrap reaches. My context held only darn's.
- Grepped `verify_attempts` and `DarnVerifyAttempts` (two calls): to check whether the stamped attempts reach the decorator's captured `Attempts`. They are stamped into the recipe before the gate is resolved, so this is not a finding.
