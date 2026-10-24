```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the `ReadAgentActivity` change forces outside the lines it touches.

**Per change, the set found:**
- The `ReadAgentActivity` behavior (a waiterless running record now logs): the callers are `internal/loomcli/decision.go:653`, `internal/loomcli/looparm.go:159` and `internal/battencli/wire.go:614`. All three only consume the readings. The returned set, the notice text (`internal/battenshed/notice.go:379`) and the run record's shape are unchanged. Covered.
- The `ReadAgentActivity` doc comment and the package doc: the new sentence is in `activity.go:57` and the new lines are in `internal/shuttleengine/doc.go:426-427`. Both match the code. `internal/battenshed/doc.go:23` describes only the fallback notice and stays correct. Covered.
- `agentwait.go:28`, the doc of the sibling reader `ReadAgentWaits`, says a run that is not live is skipped, with no logging. That reader is untouched by the diff, so the line stays correct. Covered.
- The new `waiterlessRuns` set and the `waiterlessRecovered` message: nothing else in the repo asserts or reads them. The only test that reads them is the new subtest, which uses the shared helper `requireFailureEpisode`. Covered.
- `PATTERN-log-levels`: held (the new episode logs Warn once, then Debug, then Info at recovery). `PATTERN-tolerant-records`: held (no change to `RunState`). `PATTERN-shuttle-provider-seam`: held (no provider specifics were added).
- Fixtures and existing tests: the other `ReadAgentActivity` subtests use `live` or non-running outcomes. They assert no log lines, so the new Warn lines cannot break them.
- Stencils: the diff touches none. The "Stencil wording" rule does not apply.

**Findings:** none.

**Tool calls:**
1. `grep` for the notice text, "skipped silently" and "not live" across Go, Markdown and YAML files. The manifest lacked other docs or tests that describe how not-live runs are skipped. It found nothing the diff leaves stale.
