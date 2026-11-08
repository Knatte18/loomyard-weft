---
round: 1
focus:
  - "internal/landingshed/beamdescribe.go and internal/beamcli/describe.go: does the orch's text replace the assembled description entirely, with no 'Project decisions' or decision-record listing left on any path?"
  - "internal/beamcli/describe.go: does validation reject empty or malformed text, and are the failure paths (validation, record write, PR push, landing-record stamp) handled with no partial state left behind?"
  - "internal/beamcli/describe.go and land.go: is the member PR list still offered as an editable starting point, and does the orch or operator override of the description stay possible at any time without the entry's title and brief?"
  - "contracts/orchrole/role-beams.md: does the wording tell the orch to write the end result, to list every member PR so each step can be followed, and never to list decisions made along the way?"
  - "internal/beamcli/cli.go, deps.go, pr.go and internal/dobbyengine/announce.go: do the dependency and interface changes keep every caller consistent, and does anything outside the design's scope change?"
  - "internal/landingshed/beamdescribe_test.go and internal/beamcli/describe_test.go: does each new behavior have coverage, and does any test duplicate existing coverage or assert static prose?"
  - "docs/overview.md and the doc.go files: do the doc claims match the new behavior, with no stale mention of lym assembling the description or owning summary.md?"
---

Round 1 seed. The decision record holds only the board entry under `## Goal`, so the entry is the measuring stick.
