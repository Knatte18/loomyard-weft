---
round: 2
focus:
  - "internal/planglyph/deleteorder.go, LaterDeleteReferences and the helpers deleteOrderFindings, laterEditRegions and unrewrittenLaterEditRegions: settle whether a card that appears only in the deleting slice, not in the later slice, can ever be reported as a holder, and whether it still serves as an exemption region for references its own Edit span rewrites."
  - "Callers of LaterDeleteReferences in webster (recordbatch.go and recoverbatch.go) and ValidateDispatch: settle whether a multi-card batch where a lower-numbered card deletes a member and a higher-numbered card of the same batch still references it yields a finding naming a card outside the unbegun later cards."
  - "Tests for LaterDeleteReferences in internal/planglyph: settle whether a row pins the case of a batch with two deleting cards where the higher-numbered one references the lower one's deleted member in its own Edit span."
---

The focus entries name where to look; the reviewer still reads the whole committed diff.
