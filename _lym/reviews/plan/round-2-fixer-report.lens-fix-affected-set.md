```yaml
lens: fix-affected-set
outcome: reported
```

Fix-affected-set lens: I reran each finding's search to check the fix covers every member of its set.

Unchecked findings: none. I reran F9 and F10.

**[BLOCKING] Card 9 misses loomcli's own `intentionallyNil` table (F9's set)**
- Changed text: `_lym/plan/09-pr-review-send-back-row.md`, the **Edit:** list and the Intent line "batten leaves `Landing.PullRequest` unwired, so `battencli`'s `intentionallyNil` gains `Landing.PullRequest.ShedStatus`…".
- Summary: loomcli has a second nil-seam table, also named `intentionallyNil`, which lists every `Landing.PullRequest.*` seam. `loomcli#TestWire_EverySeamFilled` fails for a nil seam that table omits. The new `Landing.PullRequest.ShedStatus` stays nil after `loomCLI.wire`, so that test fails at card 9's gate.
- Evidence:
  - `internal/loomcli/wiring_seams_test.go:14` declares `var intentionallyNil`.
  - Lines 37–57 of that file list every `Landing.PullRequest.*` seam with the reason "Env.Landing is assembled in run.go, because its constructors open the fabric pair eagerly". Beam-only seams carry their own reason.
  - Lines 192–196 fail on any nil path the table omits.
  - Card 9 has `pullRequestDeps` fill `ShedStatus`. That function runs from run.go's Landing assembly, not from `wire`.
  - Card 9 lists `internal/loomcli#TestWire_EverySeamFilled` only under **Uses:** and does not list `internal/loomcli#intentionallyNil`.
- Card 3 needs no loomcli counterpart: `loomCLI.wire` fills the drift deps through `driftDeps`, which card 3 edits.
- Neither of loomcli's other `NilSeams` callers breaks:
  - `wiring_beam_test.go:74` checks only `Pawl*` paths.
  - `wiring_darn_test.go:103` checks only the `darnSeams` it names.

**F10's set:** the only other place `captureReview` sits in a typed field is the `entries_burlerhalves_test.go:705` table. The fix leaves `captureReview` untouched, so none of these tables is affected and nothing is missing.

Tool calls:
1. Grepped every `NilSeams` / `intentionallyNil` user and every `"Drift.` / `"Landing.PullRequest.` test literal. My context held only battencli's table.
2. Read the top of loomcli's `intentionallyNil` and grepped its `Landing.PullRequest` entries and the `NilSeams` uses in the beam, darn and seams tests, to see how each one treats a nil seam.
3. Read the `NilSeams` assertion blocks in `wiring_beam_test.go`, `wiring_darn_test.go` and `wiring_seams_test.go`, to see which of them fail on a new nil seam.
