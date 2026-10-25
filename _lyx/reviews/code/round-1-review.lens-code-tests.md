```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it:

| Behaviour | Pinning test | Would it fail if broken? |
|---|---|---|
| A done run takes no slot under `max_runs` (`drivePasses` filter feeding `runsInFlight`) | `starts_test.go:181` ("done runs take no slot, get no pass and leave the state record"): two done runs, `maxRuns` 1, task `a` must start. | Yes. Without the filter, `runsInFlight` is 2, so `a` is not started and the `spawned` check fails. |
| A done run leaves no `Runs` entry, even with `owesAll` set and a stale `full_pass_owed` entry | Same test, `h.stateRecord().Runs` must be empty (`starts_test.go:~203`). | Yes. The `owesAll` branch in `recordOwed` would otherwise re-add `x` and `y`. |
| A done run gets no pass | Same test, `spawned` must be only `a`. | Only weakly. `planPass` already returns `passNone` for any non-running state, and `passes_test.go:32` ("HaltedGivesNone") pins that. The assertion is partly redundant, but it is a cheap line inside a row that needs it for the other checks, so I am not flagging it. |
| A done run with a pass in flight keeps its slot | `starts_test.go:208`: `Fatalf` if `a` starts while `r` is done and its pass is held. | Yes. Dropping the `!inFlight` guard frees the slot and `a` starts. |
| After the exit is read, the slot frees and the entry is dropped on the same tick | Same test, last tick: `a` is spawned once and `Runs["r"]` is absent. | Yes. It covers `passExited` writing the entry and `recordOwed` then removing it. |
| Done run's state entry is kept while its pass is in flight | None. | Not asserted. The decision record says the entry stays until the exit is read, but the test checks only the slot before the exit and only the drop after it. |
| Board tidy landing count and beamcli listing unchanged | None added, and none needed. | The diff does not touch that code. |
| `passes_test.go` `stateRecord` helper split | Used by the new tests. `stateOf` keeps its callers. | Not applicable. |

**Findings**

1. **LOW / consistency** — `internal/dobbyengine/starts_test.go:208-228`: the in-flight subtest never asserts that the done run's `Runs` entry is still present before the exit (after the second `h.tick(true)`). The decision record, "A done run with a pass in flight stays until the pass exits", promises the entry is kept. Dropping the entry early while the slot stays held would pass this test. Suggested fix: add `if _, kept := h.stateRecord().Runs["r"]; !kept`, or drop the claim from the doc. Evidence: `starts_test.go:213-227`, `decision-record.md`, "A done run with a pass in flight stays…".

2. **NIT / consistency** — `internal/dobbyengine/starts_test.go:181-227`: both subtests test `drivePasses` filtering and state-record behaviour, but they sit inside `TestStartReady_RefusalsAndBackoff`, whose other subtests cover refusals and backoff. `passes_test.go` already has the neighbouring "a run dropped from the listing is no longer passed" subtest (line ~333). The decision record allows "a row or subtest in the existing dobbyengine tests", so this is not a violation, only placement.

**Test economy**
- Neither new subtest duplicates an existing one. The nearest, "a run marked owned elsewhere leaves a slot under max_runs free" (`starts_test.go:158`), pins `OwnedElsewhere`, not `done`.
- "A run dropped from the listing is no longer passed" (`passes_test.go:333`) pins the generic unlisted-run drop, not the `done` filter.
- No per-helper or stencil-prose tests were added.
- The tests run on the injected clock via the harness, and both use `t.Parallel()`, which fits `PATTERN-test-speed`.

**Tool calls (3 of 6)**
1. `grep` over `starts_test.go` and `passes_test.go`: the manifest did not hold the existing test names or the harness fields (`hold`, `open`, `spawnsOf`).
2. `sed -n` on `starts_test.go:155-180`, `passes_test.go:85-125`, `passes_test.go:330-350` and `passes_test.go:20-40`: to read the neighbouring subtest, the fake's `spawn`, the dropped-run subtest and the `planPass` rows. The manifest held none of them.
