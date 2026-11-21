---
round: 2
focus:
  - "internal/websterengine/triage.go writeTriageFrictionNote: does a note whose failures are all verify command failures omit the flaky-test header, and does a test pin that by exact equality?"
  - "internal/websterengine/triage.go writeFailureOutput: does the fence stay closed when a tail line holds three or more backticks, and does a test pin the fence length?"
  - "internal/websterengine/verifygate.go newVerifyGate rerun-pass branch: does any log line or note still report a verify command failure under the name flaky?"
  - "contracts/specs/webster-spec.md, internal/websterengine/doc.go, triageWarnings' doc and contracts/stencils/landing/landing-template-describe.md: does each describe a verify command failure the way the code classifies it (no failing test or package named)?"
---

Focus restates the round's open design and consistency sites for a fresh read.
