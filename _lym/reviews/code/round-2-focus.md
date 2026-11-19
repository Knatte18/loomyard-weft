---
round: 2
focus:
  - "internal/stencilstore Read and readInclude: settle whether any path still returns embedded registry bytes to a renderer, given PATTERN-stencil-ownership says prompts are read at call time and never from embedded bytes outside the orch role."
  - "internal/stencilstore skew refusal: settle whether the error names the stencil, the writer's revision, the running revision and a way forward, and whether it fires for both untouched and edited board copies a newer binary wrote whose body differs from the shipped default, without ever writing the board file."
  - "internal/stencilstore tests: settle whether the skew behaviour is exercised through Read, ReadForEngine and ReadSet rather than an unexported helper, including the equal-commit-time, no-recorded-writer, shipped-body, no-running-commit-time and unshipped-name boundaries."
  - "internal/stencilstore writer ordering: settle whether the read path orders writers the way fabricengine.StencilSource's refresh does, ancestry first and commit time otherwise, so a descendant binary with an equal or earlier commit time is detected."
  - "contracts/stencils Registry doc comment: settle whether it still claims stencilstore.Read needs no registry."
---

Focus targets the code the round-1 fixes touch, since the fixes are new text no fresh reviewer has seen.
