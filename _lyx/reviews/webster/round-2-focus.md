---
round: 2
focus:
  - "internal/seatengine/engine.go, the advisor loop in Engine.Run and startSeat: settle whether an advisor whose strand took a held name and could not be stopped ends the run with an error wrapping ErrSeatNotStopped and its way forward, with the already-started advisors stopped, rather than leaving a live strand with no handle."
  - "internal/shedadapters/multillm.go, errorExit: settle whether the ErrSeatNotStopped path over a cancelled context agrees with card 9's wording and is reached by a test in TestMultiLLMProducer_ErrorExits."
  - "internal/seatengine/doc.go and internal/shedrecipe/entries_multillm.go: settle whether the doc.go sentence about who reads ReservedMarkers matches the code."
---

The entries restate the sites of round 1's open findings so a fresh reviewer re-checks them against the current tree.
