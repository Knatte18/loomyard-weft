---
round: 1
focus:
  - "internal/loomcli/arbiterprompt.go `arbiterVerdictGate`: does it run `ReadArbiterVerdict` on the attempt's own verdict path, pass an absent file, and carry the pass-on-cap budget the summary claims, so a still-unreadable verdict reaches `conclude` with cause `unparseable`?"
  - "`sheddriver.AgentShuttle` `Start`/`AttachIfLive`, `RunnerShuttle` and `Runner.AttachIfLiveGated`: does every caller and implementer of the changed seam pass the spec, does the tackler stay ungated with an empty `GateSpec`, and does the attach path gate the same way as the start path?"
  - "`loom-template-arbiter.md`: does the block-scalar example parse through `ReadArbiterVerdict`, and does the diff drop, weaken or contradict any stencil rule the board entry did not ask to change?"
  - "Docs in the same diff: do `internal/loomcli/doc.go` and `internal/shuttleengine/doc.go` match the code, and is any `PATTERN.md` invariant or `docs/overview.md` change owed by the new seam?"
  - "Tests: does `TestArbiterVerb_VerdictGate` duplicate coverage in `TestArbiterVerb_VerdictOutcomes`, and is any behavior added by the diff left uncovered, such as the re-prompt text or the attach path?"
---

Seed from the board entry (issue #754) and the change description, checked against the rubric's `Do not flag` list.
