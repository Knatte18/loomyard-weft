```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked what the diff's changes force outside the lines they touch.

**Per change, the set found and whether it is covered**

1. **`loomengine.ClassifyHalt` takes a fourth `buildSkew` parameter.**
   - Callers: `internal/loomcli/halt.go:72` (updated) and `internal/loomengine/anomaly_test.go:240` (updated). No other caller exists.
   - Covered.
2. **New anomaly kind `AnomalyBuildSkew = "build-skew"`, a new `anomaly:` value in the `loom-halt` note.**
   - `internal/haltnote/haltnote.go:38-65,124`: writes and parses `Anomaly` as an opaque string. Covered.
   - `internal/reportengine/stops.go:150`: `noteStop` uses the value as an opaque stop class. No closed class set, and `failureClass` derives from the reason, not the kind. Covered.
   - `selfreportengine`, `selfreportcli`, `frictionengine` and `friction` hold no anomaly filter. The reflection stencil is the only place a `loom-halt` note turns into an issue. Covered.
   - The kind list doc in `internal/loomengine/anomaly.go:17-37` is updated. Covered.
   - `docs/overview.md:1354` describes the halt notes generically ("name their anomaly kind"). Covered, needs no edit.
   - Specs: neither `contracts/` nor `_bolt/_lym/specs` names any anomaly kind. Nothing to update.
3. **`haltNote.BuildSkew` field, set by `failedHalt`.**
   - Readers: `writeHaltNote` (updated).
   - Writers: `failedHalt` (updated). The blocked-halt literals in `loomAfterStep` (`halt.go:142`) and `loomPostRun` (`arm.go:584`) leave it false, which is right for blocked halts.
   - Both `failedHalt` callers (`halt.go:140`, `arm.go:580`) pass the original step or run error, so the `errors.As` sees the chain. Covered.
4. **`friction-template-reflection.md` gains a `build-skew` drop rule.**
   - It ships embedded from the same file (`contracts/stencils/stencils.go:543`), so no second copy needs updating.
   - No other stencil reads `loom-halt` notes.
   - The earlier drop and file rules are intact.
   - Partly covered: the stated reason is incomplete (finding 1).
5. **`internal/loomcli/doc.go` new paragraph.**
   - Same incomplete reason as the stencil (finding 1).
6. **Repository rules, commit message and PATTERN.**
   - The directive declares no PATTERN changes, so there is nothing to land.
   - The commit carries no `Pattern-Departure:` trailer.
   - The `Root-Cause:` trailer is present and names the classification gap that the diff removes.
   - The `loomcli` → `stencilstore` import already existed (`darnwiring.go:356`), so no leaf-package or boundary rule is touched.

**Findings**

1. **Severity:** LOW. **Class:** consistency.
   - **Location:** `contracts/stencils/friction/friction-template-reflection.md:48-49` and `internal/loomcli/doc.go:13-14`.
   - **Summary:** Both give the reason as "the driver hands such a run to a fresh driver of the newer build". That is true only when the on-disk build changed and no hand-over was already recorded for the stop.
     - Otherwise the driver hands the run back to its parent, as the diff's own `AnomalyBuildSkew` doc says at `internal/loomengine/anomaly.go:31` ("or back to its parent to update the binary").
     - The change description says the same in its second design decision.
     - So the stencil and the `loomcli` package doc contradict the kind's own definition.
   - **Evidence:** `internal/sheddriver/driverules.go:264` (the hand-over needs `facts.BuildChanged && !facts.HandedOver`); `internal/loomengine/anomaly.go:30-33`.
   - **Fix:** word both reasons the way `anomaly.go` does: the driver hands the run to a fresh driver of a newer build, or back to its parent to update the binary, so no producer is at fault.

**Tool calls**

1. Grepped docs, internal, contracts and PATTERN for anomaly-kind descriptions and the reflection stencil's embed site; the manifest had no doc or embed search.
2. Grepped `contracts` and the hub specs for kind names, and read `reportengine`'s `failureClass` and Failures counting; the manifest held only `noteStop`, not how failures are classed.
