---
round: 1
focus:
  - "Scope against the board entry: the entry asks to find what removes a live chair strand from reed's table during Parts-Plan-Write and to keep the chair tracked while its part writers run. Settle whether the diff, which only changes how Wait reads a strand its own handle's Stop removed, addresses that cause or a different one."
  - "internal/shuttleengine/wait.go checkLivenessTick: settle whether the new `markedStopped` branch can mask a strand lost for another reason, for example a Stop on the handle that failed after marking, or a stop mark set while the strand is still live, and whether the order against the output-files check is right."
  - "internal/shuttleengine/stop.go Stop and markedStopped: settle whether `stopMarked` is set only when the strand removal ran, and whether the new lock read in markedStopped can deadlock or race with recordMu held by Wait or finalize."
  - "internal/shuttleengine/stop_test.go: settle whether the new test covers the new Wait branch through the public surface, and whether it duplicates an existing Stop or Wait liveness test."
  - "completionsignal_enforcement_test.go: settle why its lines changed and whether that change is licensed by the board entry."
  - "doc.go and the wait.go doc comments: settle whether each claim about OutcomeDied, finalize and the stopped record matches the code."
---

Round 1 seed from the committed diff of one commit over `51bd6281`; the decision record holds only the board entry under `## Goal`.
