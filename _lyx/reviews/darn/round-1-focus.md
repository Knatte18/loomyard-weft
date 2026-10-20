---
round: 1
focus:
  - "Gate wiring on both arbiter paths: does the gate reach the arbiter's start and its attach to a live arbiter, and does `AttachIfLiveGated` keep `AttachIfLive`'s rule of removing nothing on any path?"
  - "Gate semantics: does an absent verdict pass, does an unreadable one re-prompt with the reader's error and path, and does the budget cap hand on to `conclude` with cause `unparseable` as before?"
  - "Signature change of `sheddriver.AgentShuttle` `Start` and `AttachIfLive`: is every implementer and caller updated, and does the tackler's empty `GateSpec` leave its run unchanged?"
  - "Arbiter stencil `loom-template-arbiter.md`: against the board entry, does the diff drop, weaken or contradict any existing stencil rule while adding the block-scalar instruction?"
  - "Docs in the same diff: do `internal/loomcli/doc.go` and `internal/shuttleengine/doc.go` match the code, and does the change description's claims about the attach-path tests in `gate_test.go` and `arbiter_test.go` hold?"
  - "Test economy: does any new test duplicate an existing gate or arbiter test, and is any added behavior left uncovered?"
---

Seeded from the board entry (#754: make the arbiter's verdict format unable to break parsing, with a stencil rule and a check before exit) and the change description.
