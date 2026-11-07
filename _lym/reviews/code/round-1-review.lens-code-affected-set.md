```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change: the set found, covered or not**

1. **`Drift.Head` is now required by the `drift` gate.**
   - The set is `shedrecipe/entries_gate.go:279-283`, its doc comment, `shedrecipe/fixture_test.go`, `envkit.driftDeps` (sets `Head`), `loomcli/driftwiring.go:142` and `battencli/wire_seams_test.go:160`.
   - `driftcli/wiring.go` builds `Deps` for `lym drift check`, which calls `Check` and never the gate, so it needs no `Head`.
   - Covered.
2. **`{meta_dir}` placeholder.**
   - The set is `config.go` (the constant and the `CommandStep` doc), `steprun.go` render and mkdir, `capture.go`, `compare.go`, `paths.go`, `template.yaml` for `capture` and `compare`, and `doc.go`.
   - `config.go:284` still requires only `{baseline_dir}`, so `{meta_dir}` stays optional as the doc says.
   - `configcli/doc.go:118` shows an example without `{meta_dir}`, which is still valid.
   - Covered.
3. **`expected.yaml` takes `metadata`.**
   - The set is `ParseExpectation`, `Merge`, the stencil, the format constants in `seats.go`, `amend.go` (`parseAmendedEntries` goes through `ParseExpectation`), `Check`, `verify.go` and `doc.go`.
   - The sole-parser rule holds. `PATTERN-sole-parsers` names `ParseExpectation` as the only reader, and the diff adds no second parser.
   - `PATTERN-tolerant-records` holds: `ScopeRecord.Metadata` is an additive `omitzero` key.
   - Covered.
4. **Gate `Terminal` path.**
   - The set is `wait.go:1868-1884` (rejects Terminal with `pass_on_cap`; the `drift` entry has `attempts: 3` and no `pass_on_cap`), `gateattempts.go:33` (carries `Reason`), and `amend.go:33` (`amendRows` includes `darnWriteRow` and `websterRow`, so `lym drift amend` is admitted at the blocked writer row).
   - Covered, apart from finding 1 below.
5. **Landing verdict.**
   - The set is `Verdict`, `renderVerdict` and the `doc.go` verdict paragraph.
   - Covered.
6. **Rule documents and departures.**
   - The lens's declared-PATTERN-changes check finds none declared.
   - The commit message carries no `Pattern-Departure:` trailer and needs none.
   - No `docs/` file besides `docs/research/**` (measurement-style reports) names the placeholders or `volatile_*`.
   - Covered.

**Findings**

1. **LOW, class `consistency`.** Location: `internal/shedadapters/multillm.go:241` with `internal/driftengine/check.go:396-398`.
   - A terminal drift-gate failure at a writer seat becomes a Stuck reason of `reason + "; " + writerResumeWayForward`.
   - So the parent's one stop carries two way-forwards. The first is `lym drift amend <slug> --entries <file> ... then lym loom resume`.
   - The second is `re-run the row with lym loom resume, which starts every part over unless the chair is still live`.
   - After the amend, the first way forward ends in a `lym loom resume` that the second says restarts the writer's parts, which the entry meant to avoid.
   - Evidence: `multillm.go:221-241` and `check.go:309-311`.
   - Fix direction: word the unchangeable way-forward so the appended resume text does not contradict it, or have the writer exit not append its generic way forward to a terminal gate reason.
   - I did not trace the driver or arbiter path that follows a Stuck, so whether the parent also gets a second message from the driver is not settled here.
   - The handler should decide whether the one-stop focus question needs that trace.

**Tool calls (6 of 6)**
1. Grep for `driftengine.Deps{`, `Drift.CaptureCurrent`, `{baseline_dir}` and `volatile_*` consumers. My context lacked the non-driftengine wirings and docs.
2. Grep of specs, stencils and docs for expectation-format descriptions, and of config validation for placeholders. My context lacked these.
3. Grep for `Drift.Head` in seam tables, plus `envkit.go` and `driftcli/wiring.go`, to check the `Head` seam in the other `Deps` builders.
4. Grep for `gateTerminal` in `shuttleengine` and for `Reason` in `shedadapters`. This traced how a Terminal failure reaches the parent.
5. Read of `multillm.go:222-243` and `writerResumeWayForward`. My context lacked what the writer exit appends to a gate reason.
6. Grep for `amendRows` in `amend.go`, to confirm `lym drift amend` is admitted at the blocked writer row.
