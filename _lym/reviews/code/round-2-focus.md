---
round: 2
focus:
  - "internal/gatefixer/rowgate.go, waitOnly and waitThenRun: when does the fixer record turn terminal? Settle whether a clean earlier entry that passes over the fixer's finished tree leaves the record live for a later entry (Darn-Write's verify) that fails, so that failure carries the fixer's reply, and whether a driver restart between the two entries still finds the record live. Also settle whether an erroring inner closure leaves the record live."
  - "internal/gatecli/detachfixer.go, runWithFixer and fixAndRerun: which first exits reach a fixer, and where the fixer record is marked terminal relative to the re-run. Settle whether a refusal (exit 1, JSON envelope on stdout only, per the exit-code section of internal/gatecli/doc.go) can spawn a fixer, and whether a restart during the re-run still finds the record live."
  - "internal/gateslot/config.go and internal/gateslot/tiers.go: every occurrence of the card gate key name, against PATTERN-named-constants (one named constant, declared once, referenced everywhere). Also internal/loomcli/gatefixer.go gateFixerBodies keys against the entry-name constants beside them."
  - "Test coverage of the diff's new guard wiring: writeNewRun writing the caller's role into the run request that handsOff reads (internal/gatecli/detachfront.go), and the three loomcli goto sites (internal/loomcli/drivewiring.go, arm.go, arbiterconsume.go) wrapping their inner guard with fixerEndingGuard. Settle which of these a test pins and which a removal would leave green."
---

Fixes applied after round 1 are new text; review them as part of the diff.
