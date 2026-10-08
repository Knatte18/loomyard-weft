# Card 18 — stuck-outcome-pin

**Edit:**
- `internal/websterengine/runlevel_test.go`
- `internal/shedadapters/webster_test.go`

**Intent:**
Pin that a Master's stuck outcome is never reported as a question (D9); no production code changes, since `shuttle-turns` removed the path that produced `webster master is asking a question`.

`runlevel_test.go`'s `TestRun_DoneOutcome` gains a row whose Master writes `outcome.yaml` with `outcome: stuck` and a `stuck_reason`, plus `summary.md`, and finishes shuttle-done: the `RunResult` reads `stuck` with that reason, and neither the reason, the summary title nor any warning the run returns contains `question`.
`webster_test.go`'s `TestWebsterProducer_OutcomeStuck` additionally asserts that the Stuck pointer's reason is the stuck reason verbatim and contains no `question`.

**ImpactSummary:** Test-only regression pin on webster's stuck-outcome path.
