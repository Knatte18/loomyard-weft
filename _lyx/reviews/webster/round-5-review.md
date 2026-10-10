---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/hubgeom/webstergeom_test.go:106
    summary: No test reaches the gate.yaml way forward through a Go-side gate site; the hubgeom absent-file row pins only the file name, and slots below 1 is reached through no site
  - id: F2
    severity: LOW
    class: decision
    location: internal/planglyph/typesload.go:138
    summary: The plan gate's type load swallows a gate slot acquire failure into the import-path scan with a log line, so an unusable gate.yaml gets no refusal, way forward or spec row at this one slotted site
  - id: F3
    severity: LOW
    class: scope
    location: internal/webstercli/validate.go:108
    summary: No test pins that the whole-plan validate verbs call the wired index; reverting scopedValidate or loomcli validate-plan to planglyph.Validate would pass every test
  - id: F4
    severity: LOW
    class: consistency
    location: internal/planglyph/glyphchain_integration_test.go:437
    summary: The "plan gate pass" slotted row repeats what the "whole-plan Validate through a slotted index" row covers, since Validate runs planGatePass through the same loader
---

Range: `product.parent` is `main`, no `_lyx/loom/rework/` exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `10df04b02`).
I read the diff statically.
I ran no tests: the `.dev-bin/lyx` is older than the round-4 fixes and has no usable `gate.yaml` in this hub, so a slotted run would not have exercised HEAD.

### [MEDIUM] No test reaches the gate.yaml way forward through a Go-side gate site

**Location:** `internal/hubgeom/webstergeom_test.go:106`, `internal/websterengine/cardverify_test.go:69`, `internal/verifytree/verify_integration_test.go:299`

**Issue:**
`PATTERN-refusal-way-forward` requires a test to reach each refusal, and the two `gate slot: gate.yaml absent or invalid` rows in `contracts/specs/refusal-spec.md` name `lyx fabric reconcile` and `lyx config gate` as the way forward.
The message text is pinned only at `gateslot.LoadConfig` (`internal/gateslot/config_test.go:39-42`) and at `lyx gate test` (`internal/gatecli/test_integration_test.go:396-423`).
At the sites the rows bind:
- `hubgeom.GateSlots` is reached with an absent file, but the assertion checks only that the error contains the file path; it does not check `lyx fabric reconcile`, and no row uses a `gate.yaml` holding `slots: 0`.
- The card-verify rerun is reached with a fake `Limits` error (`gate.yaml unreadable`), so the way-forward text never appears in its expected failure line.
- `verifytree.Verify` has no test where `Acquire` fails, so the `verifytree: wait for gate slot:` wrap, the marker cleanup on that return and the verb's `webster: verify:` envelope are unreached.

**Fix:**
Make the `hubgeom` absent-file check also require `lyx fabric reconcile`, and add a row with a `gate.yaml` holding `slots: 0` that requires `lyx config gate`.
Add one table row to `TestVerify_SlotGate`'s neighbour (or a small table beside it) where `Limits` returns the real `LoadConfig` error and `Verify` returns an error containing the way forward with no marker left behind.

### [LOW] The plan gate's type load swallows a slot acquire failure

**Location:** `internal/planglyph/typesload.go:138`, `internal/planglyph/typesload.go:66`

**Issue:**
`goListLoader.load` returns the `Acquire` error, but `loadTypedReferences` treats every loader error as "no type information" and answers from the scan (`doc.go` states a failed load is "never an error").
So when `gate.yaml` is absent or invalid, every other slotted site refuses naming `lyx fabric reconcile` or `lyx config gate`, while the plan gate and the whole-plan validate verbs silently drop to the scan and leave only a `logger.Warn`.
The scan marks a name-only method match informational, so a caller the type load would have resolved as blocking can pass.
Card 09 says the wait record is removed "held or failed" but leaves the failure's disposition open, and `refusal-spec.md` has no row for it.
Either answer is defensible; the choice is open.

**Fix:**
Decide one of:
- Return a gate-infrastructure error from `planGatePass` when the acquire fails, naming the way forward, with a spec row in the loom section and a test row.
- Keep the degrade, state in `doc.go` and the spec that an unusable `gate.yaml` degrades the plan gate to the scan, and cover it with a row where `Limits` fails and the findings equal the scan-only answer.

### [LOW] No test pins the validate verbs to the wired index

**Location:** `internal/webstercli/validate.go:108`, `internal/webstercli/validate.go:121`, `internal/loomcli/validate.go:189`

**Issue:**
Round 4's F1 moved three call sites from `planglyph.Validate` to the wired index so the type load runs in a slot.
`internal/planglyph/glyphchain_integration_test.go` covers `NewSlottedIndex(...).Validate`, but the webster and loom tests wire the unslotted `planglyph.NewIndex` (`internal/webstercli/cli_test.go:142`, `internal/loomcli/parity_test.go`), so a revert to `planglyph.Validate` at any of the three sites changes no test result.
`PATTERN-gate-slot` has no scan, so review and tests are its only guard.

**Fix:**
Give the webster and loom CLI tests a stand-in `planindex.Index` that records its `Validate` call, and assert `lyx webster validate` before a batch has begun and `lyx loom validate-plan --require-approved` each call it once; fold these into the existing validate tests as rows.

### [LOW] Redundant slotted row for planGatePass

**Location:** `internal/planglyph/glyphchain_integration_test.go:437`

**Issue:**
`slottedEntries` runs the same hold, wait-record, release and findings-equality scenario through `planGatePass` and through `NewSlottedIndex(...).Validate`.
`validate` calls `planGatePass` with the index's own loader, so the `whole-plan Validate through a slotted index` row (line 446) covers everything the `plan gate pass` row covers, and it is the public-surface one.
The plan-gate row is the per-helper test `PATTERN-test-economy` asks to fold away, and each row waits on real `go list` runs.

**Fix:**
Drop the `plan gate pass` row, or replace its entry with `NewSlottedIndex(...).ValidateFormat`, whose path differs from `Validate`'s, so each remaining row covers a distinct entry.

### Judged against the focus directive

1. Whole-plan validation: every production path that reaches the plan gate's type load goes through a slotted index.
   `scopedValidate` (both branches) and `loomcli validate-plan --require-approved` call the wired index's `Validate`; the gates, the `Webster` geometry and the loom `wire` and `wireLightweight` paths build `NewSlottedIndex`; `planGatePass` is reached only from `validate`, `validateFormat` and `validateFormatAfter`, which take the index's loader.
   `defaultTypesLoader` is read only by the exported `Validate`, `ValidateFormat`, `ValidateFormatAfter`, `ValidateRework` and by `NewIndex`, none of which has a production caller; card 9 keeps them, so this is not a defect.
   Standalone webster passes a nil pool and loads unslotted, as card 9 and `doc.go` state.
2. Acquisition failures at `verifytree.Verify`, the card-verify rerun and `hubgeom.GateSlots`: the returned errors carry both ways forward through `%w` (`LoadConfig`, then `GateSlots`, then `Pool.Acquire`, then `Verify`), and the webster and loom spec rows exist.
   The gap is test reach, F1, and the one site that does not refuse, F2.
