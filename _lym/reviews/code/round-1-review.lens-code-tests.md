```yaml
lens: code-tests
outcome: reported
```

**Per behaviour → pinning test**
- Finalize merges through the resolver under the parent lock for every recipe, with no abort and no overlap note: `TestFinalize_Merge` rows in `finalize_merge_test.go` plus the integration test's single call that ends Done.
- A conflict ends Done with one session and no reroute: `finalize_integration_test.go`.
- An overlapping parent change lands with the overlap recorded: the "overlapping parent change lands" row (`wantOverlap`).
- The unreadable and absent merge-record fallback: the "unreadable" and "no merge record" rows.
- The retry after a merge-in-required error merges, gates and lands a code-side commit: the new `parentMoves` row in `finalize_test.go`.
- Escalate prefix and way-forward text in `haltReason`: `fixerrun_test.go`.
- Finalize escalate prefix on loom or darn, and pawl keeping `LandingStopPrefix` with no decision verb: `finalize_gate_test.go`.
- Parent-Align escalate prefix, and pawl: `parentalign_fixer_test.go`.
- `EscalateHalt` and `HandsBack` over the three rows and another row: `handback_test.go`.
- `arbiterstop.Check` admits an escalate halt: `arbiterstop_test.go`, two new rows.
- `EscalateJudgedFirst` per recipe: `wiring_beam_test.go` and the `landingDeps` drift-guard in `landingdeps_test.go`.
- `JudgesFirst(producer, reason)` in loom's seam: `TestLoomArbiterSeam_JudgesFirst` in `drivewiring_test.go`.
- The driver reads Reason for a step-built stop and Error for a status-built stop, and a plain reason still meets the tackler: `drive_test.go`.
- `restep_hold` in the loop envelope: `loop_test.go`; in status, including no check told: `status_test.go`.
- `specFor` tells `RestepHold` for loom and darn only: `TestSpecFor_RestepHold` in `cli_test.go`.
- Batten `restepSkips` and the wired `InnerRun.HandBack`: `TestRestepSkips` and `wire_seams_test.go`.
- Darn's `ReviewInput` keeps the shared seam: `wiring_darn_test.go`.
- Driver stencil and guide `restep_hold` skip: no test, correct for static prose.
- `loomRerouteCapWayForward` text edit: no test; static text, non-finding.

**Findings**

1. NIT, class `consistency`. `internal/battencli/wire_seams_test.go`, the added `escalated` assertion in `TestWire_EverySeamFilled` partly overlaps `TestRestepSkips`' `EscalateHaltAtFinalize` row; its unique value is the wiring into `InnerRun.HandBack`.

2. LOW, class `consistency`. `internal/landingshed/handback_test.go`, the `EscalateAtMergeIn` and `EscalateAtParentAlign` rows and the two `arbiterstop_test.go` rows partly overlap the `requireEscalateHalt` assertion in `parentalign_fixer_test.go` and `finalize_gate_test.go`. Small overlap; removal not required.

3. No uncovered behaviour.

**Tool calls**
1. `git diff … -U2 -- 'internal/*_test.go'`: first look at the test diffs; truncated.
2. `git diff … -U1` over the named landingshed, sheddriver, shedverbs and loomcli test files.
3. `sed` on the saved diff output plus a `git diff` of `drivewiring_test.go`, `wiring_beam_test.go`, `wiring_darn_test.go`, `wire_seams_test.go` and `loop_test.go`.
