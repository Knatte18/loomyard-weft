---
round: 1
focus:
  - "internal/driftengine/judgesnapshot.go: does the decision-record exemption cover only commits from `lym loom decision add --by parent`, and still catch a judge-made change to the record or any other file?"
  - "internal/driftengine/judgesnapshot.go: do the failure paths (unreadable record, missing snapshot, unparseable entries) fail closed, and does the new dependency in deps.go, seats.go and internal/loomcli/driftwiring.go wire the same way in every caller?"
  - "internal/driftengine/expectation.go: the diff refuses a line range over a case whose every line is keyed, where the issue asks for real line numbers or key-based entries; is the refusal correct on mixed cases, empty cases and `diff: lines` vs key reporting, and does it settle issue #141 for Drift-Expect's output?"
  - "internal/driftengine/judgesnapshot_test.go and expectation_test.go: do the new tests cover the exemption's boundaries and the refusal's boundaries, without duplicating each other?"
  - "internal/driftengine/doc.go and the commit messages: does each claim match the diff, and do the docs the repository's constraints name for this kind of change land in the same diff?"
---

Round 1 seed. The answer key is the decision record alone, with the Goal's two issue parts as the measuring stick.
