# Card 5 — webster-expired-shells

**Edit:**
- `internal/websterengine#Run`
- `internal/websterengine/summary.go`
- `internal/websterengine/triage.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/summary_test.go`

**Uses:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#Result`
- `internal/websterengine#AppendIntegrationTriage`
- `internal/websterengine#writeTriageFrictionNote`

**Intent:**
D1's Webster declaration and D2's Webster surface.

Master's spawn in `Run` sets `Spec.AwaitedShellPrefixes` to one named constant, `lyx webster recover-batch`:
the failure ladder backgrounds that verb, and `recovery_timeout_min` already bounds it, so the gate waits on it like a fork.

After Master's wait, every label in `Result.ExpiredShells`:
- is written, whatever the outcome, into one Go-authored friction note (`webster-background-shell`, beside `writeTriageFrictionNote` in `triage.go`, best-effort and a no-op without a friction directory) naming each shell and the bound, so reflection has the evidence a hang otherwise leaves only in `events.jsonl` (PATTERN-friction-capture);
- on a done outcome, adds one warning to `RunResult.Warnings`: "turn end counted after background shell `<label>` ran past `background_shell_wait_min`; the shell may still be running in the session";
- on a done outcome with `summary.md` present, lands in a `## Background shells waited out` section, appended by a new `AppendBackgroundShells` in `summary.go` following `AppendIntegrationTriage` (a no-op on an empty list), one bullet per label.
A non-done outcome keeps its own error or stuck reason; the friction note and shuttle's own log line carry the labels there.

Tests: a fake Master result with one expired shell yields the warning text exactly, the summary section, and the friction note;
an empty list writes none of the three;
Master's spec carries the awaited prefix.
Document the awaited prefix and the expired-shell warning in `internal/websterengine/doc.go`.

**ImpactSummary:** Webster's Master spec gains one declared prefix and its done path one more warning source; no other outcome changes.
