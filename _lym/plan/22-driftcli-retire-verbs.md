# Card 22 — driftcli-retire-verbs

**Delete:**
- `internal/driftcli#scopeCmd`
- `internal/driftcli#acceptCmd`
- `internal/driftcli#holdAcceptLock`
- `internal/driftcli#acceptTest`
- `internal/driftcli#withdrawCmd`
- `internal/driftcli#spentRefusal`
- `internal/driftcli#takeRun`
- `internal/driftcli#compareCmd`
- `internal/driftcli#runSideCapture`
- `internal/driftcli#waiveCmd`
- `internal/driftcli#compareLockRetry`
- `internal/driftcli/accept_integration_test.go`

**Edit:**
- `internal/driftcli#Command`
- `internal/driftcli#captureCmd`
- `internal/driftcli#statusCmd`
- `internal/driftcli#readStatus`
- `internal/driftcli#deferCmd`
- `internal/driftcli_test#TestDrift_RunsInTheAnchoredDirectory`
- `internal/driftcli_test#TestCapture_OutlivesCaller`
- `internal/driftcli/doc.go`

**Uses:**
- `internal/driftcli_test#statusBody`

**Intent:** Remove the `scope`, `accept`, `withdraw`, `waive` and `compare` verbs and `capture --side` with their help, tests and wiring helpers (D14); the engine functions they called go in card 25.

`Command` registers `check`, `amend`, `capture`, `defer`, `status` and the hidden `capture-child`, and its long help describes D3's config-chosen scope, the sealed expectation and the check.
`captureCmd` loses `--side` and its help paragraph; `statusCmd` and `readStatus` drop the Drift-Verify run count and print the scope record, the capture record, the two locks and the diff mode, recorded or config's, with `compare_ignored: true` under `lines`, since `lines` never runs the compare (D2); `deferCmd`'s help no longer names `waive` and lists the halts D12 admits.
`TestDrift_RunsInTheAnchoredDirectory` runs `check` where it ran `compare` and `capture --side`, still asserting the command ran in the anchored directory; so that `check` reaches its capture, the test seeds, beside its tapestry seed, a status at Drift-Verify, past Drift-Expect, a scope record with scope `full`, `awaited_at` set and `diff: lines`, a passed `capture.json` and a baseline directory holding one case file, with no seal and no `expected.yaml`.
`TestCapture_OutlivesCaller` drops its `compare` steps, keeps the detached capture's checks, and asserts its `status` body carries `compare_ignored: true` under the template's `lines`.
The package doc lists the verbs that remain and drops the `# Runs` and `# Accept` sections.

**ImpactSummary:** Removes six verbs and their helpers from the `lym drift` tree; the CLI tree tests walk the tree and need no list change; two integration tests move to `check`.
