---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/boardcli/verbs_test.go:347
    summary: The repository sweep leaves layout goldens for the same human-read compact listing and a README heading assertion.
    fix: "In internal/boardcli/verbs_test.go, remove the spacing-sensitive wantList, wantFind and wantLabelled comparisons in stepListAndFindText and the sketches heading-layout assertion in stepListShowsASketch. Retain CLI success, filtering, sorting and entry-selection checks without exact presentation expectations, using the existing JSON checks where they cover the behavior. In internal/boardcli/cli_test.go, replace stepLoadsHubBoardConfig's literal ### Features assertion with a check of the configured type in board-data.js or the board view. Keep the file-existence and config-loading checks."
---

### [MEDIUM] The sweep leaves human-read layout expectations

**Location:** `internal/boardcli/verbs_test.go:347`

**Issue:** The goal requires a repository sweep deleting other human-read layout goldens.
Deleting `TestRenderCompact` leaves `stepListAndFindText` comparing column-padded stdout byte for byte.
Its `wantList`, `wantFind` and `wantLabelled` comparisons remain at lines 347–370.
The sketch listing check also pins its heading and line placement at lines 409–411.
`stepLoadsHubBoardConfig` still expects `### Features` in the README at `cli_test.go:1169,1206`.
These assertions make presentation changes require matching test rewrites after this cleanup.
This is an omitted part of the requested sweep, rather than a request to change unrelated production behavior.
The scope and affected-set lenses independently reported the same omission; they are consolidated here.

The retained rewrite, output-path, manifest-cleanup and age tests still assert their named behavior.
The removed standalone board-data existence assertion duplicates the retained loop in the same subtest.
The view literals cover every fixture placement formerly checked through README parsing,
and they additionally check the done-project entry's section and dependencies.
The caller-owned discussed-flag assertion survives in the view test.
The compact listing's filtering and selection checks remain in the CLI suite;
removing its unit golden does not remove their only coverage.
The layer test's keep-marker edit follows from deleting the goldens.
No production file changes, unsupported departure trailer, or merge-format change appears in the range.

Validation was attempted with `lym gate test --tier 0 ./internal/boardengine ./internal/boardcli`.
The gate could not acquire a slot because the sandbox exposes the hub's gate records as read-only.
No test ran from that attempt; this environmental refusal is not a finding against the diff.
