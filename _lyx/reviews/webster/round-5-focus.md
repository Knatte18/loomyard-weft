---
round: 5
focus:
  - "Whole-plan validation in internal/planglyph/planglyph.go (Validate), reached from internal/webstercli/validate.go (scopedValidate and the validate verb) and internal/loomcli/validate.go (validate-plan with --require-approved): does every path that runs the plan gate's type load go through the slotted index loader, with no call to defaultTypesLoader left outside a gate slot (PATTERN-gate-slot)?"
  - "Gate slot acquisition failures on the Go-side gate sites (internal/verifytree/verify.go, internal/websterengine/cardverify.go, internal/hubgeom/gateslots.go): when gate.yaml is absent or holds an invalid value such as slots below 1, does the returned error name a way forward, does refusal-spec.md carry a matching row, and does a test reach the message (PATTERN-refusal-way-forward)?"
---

Focus targets the sites the latest fixes touched, plus the scoped invariants they rely on.
