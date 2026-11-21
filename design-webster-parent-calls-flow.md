# Webster and plan fix honour parent decisions; full halt reasons

`webster-parent-calls-flow` · Task · bug, webster, burler, drift, loom · running · PR-Review

{"error":"invalid json: invalid character 'w' looking for beginning of value","ok":false}

## Triage against main (2026-11-21)

- Create check: `internal/planglyph/donecheck.go:108` reports create-not-done without reading the decision record; `createTargetWayForward` in `internal/websterengine/recordbatch.go` gives the two verbs different ways forward.
- Drift halt: `internal/websterengine/verifygate.go` cuts findings at `verifyGateStuckTextBytes` (400 bytes) and names no report file; nothing lets the parent accept expected drift.
- Plan amendment: `internal/burlerengine/roundcheck.go` refuses a finding in a file the changes file does not list, with no exemption for an `--amends-plan` decision, and Webster's halt reason never reaches Plan-Burler-Fix as a finding.
