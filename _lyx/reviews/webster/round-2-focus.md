---
round: 2
focus:
  - "In internal/creelengine/seam.go, settle whether `ExternalSender` forms the EXTERNAL program-sender name only through `agentname.FormatProgram(shortname, agentname.External)`, and whether any other non-test site in internal/creelengine still joins a shortname and `agentname.External` by string concatenation or formatting."
  - "In internal/creelengine/identity.go (`resolveSender`, `externalFromClaim`) and their callers, settle whether the error `ExternalSender` can now return for an invalid shortname is propagated and ends in a refusal, never a silently empty or malformed Sender."
  - "In internal/creelengine/identity_test.go and internal/creelengine/verbs_test.go, settle whether callers of `ExternalSender` compile against its signature and whether the refused-invalid-shortname case is covered by a row in an existing table test rather than a new top-level test."
---

Round 2 targets the committed fix for the EXTERNAL-name formation; the rest of the diff was reviewed in round 1 and stays context.
