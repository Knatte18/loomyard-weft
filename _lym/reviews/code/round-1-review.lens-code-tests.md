```yaml
lens: code-tests
outcome: reported
```

**Behaviour → pinning test**

- `RunAcceptedCapture`, pass and file left in the output directory: `capture_test.go` `TestRunCapture`, the `acceptedRows` rows 1 and 2. Pinned.
- `RunAcceptedCapture`, directory emptied first: `acceptedRows` rows 2 and 5. Pinned.
- `RunAcceptedCapture`, non-zero exit: `acceptedRows` row 3. Pinned.
- `RunAcceptedCapture`, exit 0 with no file: `acceptedRows` rows 4 and 5. Pinned.
- `RunAcceptedCapture`, no capture record written and slot freed: the trailing assertions in the `acceptedRows` loop. Pinned.
- `RunAcceptedCapture`, timeout branch (`result.Reason = "timed out after …"`): none. The only timeout test at `capture_test.go:241` covers `RunCapture`.
- `RunAcceptedCapture`, error returns (`RemoveAll` failure, `captureInto` error, `holdsFile` error): none.
- `RunCompare` `ProfileDir`: `compare_test.go`, "a told profile directory replaces the baseline directory in the command". Pinned.
- `RunCompare` with empty `ProfileDir` still renders the baseline directory: the existing `TestRunCompare` rows. Pinned.
- `Accept` and `Withdraw` clear `AcceptedProfileHead`: `verbs_test.go` `TestAcceptWithdraw`, which seeds `"aaa"` and asserts it is cleared. Pinned.
- `DriftRecheck.pass`, Done with no compare (no scope record, scope off, waived, HEAD equal to the compared head): the "a pass that needs no compare is Done" rows. Pinned.
- `DriftRecheck.Call`, drift switched off: "drift switched off is Done with no compare and no commit". Pinned.
- `DriftRecheck.pass`, Stuck with no compare (no compare recorded, dirty tree, undeclared compare), with the earlier compare left untouched: the "a pass that cannot compare" rows. Pinned.
- `DriftRecheck.pass`, no acceptances, baseline compare pass and fail, with `recordCompare` and no profile head set: the "with no test accepted" rows. Pinned.
- `DriftRecheck.pass`, accepted compare passing, which moves both heads and keeps the acceptances: "a passing compare against the accepted profile…". Pinned.
- `DriftRecheck.pass`, accepted compare failing, which records the failure and lapses: "a failing compare is Stuck naming the accepted tests and lapses". Pinned.
- `DriftRecheck.pass`, profile head empty or older than the compared head, which lapses with no compare: "tests accepted with the profile captured at another commit lapse with no compare". Pinned.
- `DriftRecheck` missing-seam check (`requireSeams`): no producer-level test. `entries_drift_test.go` covers the nil seams at entry level through the `RecheckNil*` rows.
- `keepAcceptedProfile`, capture once and record HEAD, capture again for an older head, no capture when already at HEAD, no capture when nothing is accepted: `verify_test.go`, "the accepted profile" rows. Pinned.
- `keepAcceptedProfile`, failed capture is a Stuck naming resume, records no head, and commits once: "a capture that does not pass is Stuck…". Pinned.
- `keepAcceptedProfile`, agent path through `runAgent`, captured at the agent's HEAD: "an agent's acceptance is captured at the agent's HEAD before Done". Pinned.
- `keepAcceptedProfile`, `CaptureAccepted` returning an error: none.
- `keepAcceptedProfile`, interrupted-agent resume that reaches Done: none specific. It shares the `runAgent` call site, which the agent-path test covers.
- Recipe wiring (`on_done`, `on_stuck`, `entry_gate`, row numbers, engine type, sequence skip): pinned by `darnrecipe_test.go`, `progress_test.go`, `drift_test.go`, `recipe_test.go`, `shape_test.go` and `sequence_test.go`.
- `driftRecheckEntry` and the `CaptureAccepted` seam check in `driftVerifyEntry`: pinned by `entries_drift_test.go`.
- Batten nil seams: `wire_seams_test.go`. Pinned.
- Reportengine timing spans for `NameDriftRecheck`: `timing_test.go` was edited in the diff; I didn't read it, so I take it as pinned.
- Interrupt policy for `NameDriftRecheck`: no new row seen. Whether an existing exhaustive test pins it is unverified.
- `driftwiring.go` `CaptureAccepted` and `CompareAccepted` (the `Skip: scope.Skip`, `ProfileDir: paths.AcceptedDir` and `LockRetry` settings): none. The grep found no loomcli test that exercises them; `loomcli/export_test.go` only references `driftDeps`.

**Findings**

1. MEDIUM, design. `internal/loomcli/driftwiring.go:194-221`: `CaptureAccepted` and `CompareAccepted` have no test.
   - Their request fields decide the core behaviour: accepted tests stay in the compare because `Skip` is `scope.Skip` and not the accepted-extended list, and the compare reads `paths.AcceptedDir`.
   - The `driftengine` tests stub these seams, so swapping in `SkipList` or dropping `ProfileDir` would break no test.
   - Evidence: no `*_test.go` outside the stubs references `CompareAccepted`; `recheck_test.go:72` stubs it.

2. LOW, design. `internal/driftengine/capture.go:~167`: the `RunAcceptedCapture` timeout branch, `"timed out after …m; log …"`, has no row in `acceptedRows`.
   - Add a timed-out row beside the non-zero-exit row, mirroring the `RunCapture` timeout test at `capture_test.go:241`.

3. LOW, design. `internal/driftengine/verify_test.go`, "the accepted profile": no row for `CaptureAccepted` returning an error. The pass should return `("", {}, err)` with the profile head unchanged. Add one row.

**Tool calls**
1. A grep over `*_test.go` for `CompareAccepted`, `CaptureAccepted` and `driftDeps(`. My context held only the changed tests, so I needed to know whether the `driftwiring.go` seams are tested elsewhere.
2. A grep of `capture_test.go` for `timed out` and `TimedOut`, run in the same call, to see whether the accepted-capture timeout is covered.
