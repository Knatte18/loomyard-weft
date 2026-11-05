---
round: 2
focus:
  - "internal/driftengine/check.go, NewDriftGate: settle whether the gate still fails on an unexpected cell or a failed current capture when the report also carries a records or seal fault, and whether the check_test.go table has rows combining each fault with an unexpected cell."
  - "internal/driftengine/recordsguard.go, WriteRecords, and internal/driftengine/amend.go, amendRows: settle whether every refusal that names `lym drift amend <slug> --restore` is at a row where Amend admits --restore, and whether a records mismatch at Drift-Expect or Drift-Baseline reaches the parent as a halt rather than a plain error."
  - "internal/driftengine/expect.go, runAgent and pass: settle whether an existing valid expected.yaml is sealed without a spawn only after an expect session that ended, and whether the closed session mark distinguishes that from a session never opened."
  - "internal/driftengine/diff.go, listLines: settle whether a key pattern whose key group matches the empty string is treated as no match, so that no Identity with an empty Key and a Line of 0 is produced."
---

Round 1 was judged from its review; these entries direct round 2 to the code those fixes touch.
