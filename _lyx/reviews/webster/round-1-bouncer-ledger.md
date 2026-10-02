---
round: 1
ledger:
  - key: batten-halt-warned-marker-not-cleared-on-resume
    rounds: [1]
    status: resolved
  - key: shuttle-terminal-on-pass-on-cap-entry-unguarded
    rounds: [1]
    status: resolved
  - key: parentreview-gateconfig-reviewer-doc-names-seed
    rounds: [1]
    status: resolved
  - key: fixed-column-wrapped-new-comments
    rounds: [1]
    status: resolved
---

Round 1 reviewed the full branch range over `main`'s merge base `812600b3a` (cards 1–11 plus the trailing `reedcli` naming-test commit).
It raised four LOW findings: the batten halt-warned marker surviving an observed resume, a Terminal result from a PassOnCap gate entry finalizing as a reasonless pass, `GateConfig.Reviewer`'s doc still naming the seed, and fixed-column wrapping in new `fabricengine` and `battencli` comments.
All four were fixed in one commit each and verified by the judge; no finding remains open.
