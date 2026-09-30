# Card 5 — Webster-owned "run in flight" predicate

**Create:**
- `plan:internal/websterengine#RunInFlight` -> `func RunInFlight(anchorRoot string) (bool, error)`
- `internal/websterengine/inflight_test.go`

**Uses:**
- `internal/websterengine#Dir`
- `internal/websterengine#OutcomePath`
- `internal/websterengine#parseOutcome`
- `internal/websterengine#stateFileName`
- `internal/websterengine#outcomeDone`

**Intent:**
Export the one predicate another package needs to ask whether a Webster run is mid-flight at an anchor (D5), so no other package ever constructs webster's paths (Cwd Resolution and Told-Geometry invariants).

`RunInFlight` lives in a new file `internal/websterengine/inflight.go` with its own header comment.
It builds every path from the anchor root it is handed through `Dir` and `OutcomePath` — no `lyxcwd` import.
It returns false when `state.json` does not exist under `Dir(anchorRoot)`;
true when `state.json` exists and `outcome.yaml` is absent;
otherwise it parses `outcome.yaml` with `parseOutcome` and returns true for any outcome other than `done` (`paused` and `stuck` runs will resume, so they count as in flight) and false for `done`.
A `state.json` stat failure other than not-exist, and an unreadable or malformed `outcome.yaml`, return the error.
The doc comment states why `paused` and `stuck` count as in flight and that only webster's own durable files are read, never loom's shed status.

Tests, untagged in package `websterengine_test` over `t.TempDir()` anchors (no git), one table covering: no `state.json` → false;
`state.json` without `outcome.yaml` → true;
outcome `paused` → true;
outcome `stuck` with a reason → true;
outcome `done` → false;
a malformed `outcome.yaml` (an unknown outcome value, and unparseable YAML) → error;
an unreadable `outcome.yaml` (a directory created at its path, which fails the read portably) → error.
Seeded files are written under `websterengine.Dir(anchor)` and `websterengine.OutcomePath(websterengine.Dir(anchor))`.

**Commit:** 5: websterengine: RunInFlight predicate for a mid-run Webster at an anchor
