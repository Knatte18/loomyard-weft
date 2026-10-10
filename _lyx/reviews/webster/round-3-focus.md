---
round: 3
focus:
  - "contracts/specs/refusal-spec.md loom section: does `lyx loom start` reaching `refuseOverUnfinishedMerge` in internal/loomcli/start.go have a row whose trigger names start as well as resume, and does the way forward match the message?"
  - "internal/loomcli/start.go: does every refusal that returns a bare `err.Error()` (merge-state probe, run-lock probe, strand table read, retiring-strand removal, park-marker removal, status read in `runHaltedAtHandBack`) now name a retry or file to fix, have a row in the loom section of contracts/specs/refusal-spec.md, and have a test reaching it that asserts the way forward?"
  - "internal/loomcli/start.go: do `currentProducer` and `runHaltedAtHandBack` agree on whether an undecodable status file blocks `lyx loom start`, and is the chosen rule covered by a test?"
---

The focus follows the two open findings.
