---
round: 2
focus:
  - "internal/summaryparser/description.go, mergeDangerSection and isSectionHeading: does a line inside a ``` or ~~~ fence count when matching the `## Merge danger` heading or when ending the section, and do TestValidateDescription_Table rows cover a fence-only section, a fenced example before the real section, and a `#` line in a fenced block inside the real section?"
  - "internal/heddleengine/glossary.md: does any term line name a storage location, file name, path or runtime host that the board entry asks the glossary to leave out?"
  - "internal/beamcli/cli.go, doc.go and describe.go: do the `describe --draft` help, the package doc and the draft envelope's way_forward tell the orch to call the Merge danger door and blast radius before recording, as RecordBeamDescription requires?"
  - "internal/landingshed/beamdescribe_test.go and internal/loomengine Discussion tests: is the unedited beam draft recorded and refused, and does a test assert the Discussion prompt carries the glossary text?"
---

Focus restates the sites where round 1's fixes land, so fresh reviewers check them against the code and not against a prior review.
