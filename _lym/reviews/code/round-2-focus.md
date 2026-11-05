---
round: 2
focus:
  - "internal/battencli/startguard.go refuseAtStart and internal/battencli/arm.go start path: settle whether a `lym batten run` or step of a raddle entry over a done motif run removes or pushes the run record only after the owner-machine check and every start-guard refusal (raddle, unreleased, dirty-source, left-pair, cancel marker) has passed, and whether a refused start leaves the done run directory in place."
  - "internal/battencli/startguard.go clearMotifRunOfRaddleEntry and startClearsMotifRun: settle whether the board entry's recipe is resolved the same way the guard's raddle refusal resolves it (entry recipe or board default), and whether a test reaches the closure that gates the clear, including a slug absent from the board, an unreadable board, a non-raddle entry and a raddle entry whose done run's child ran raddle."
  - "internal/fabricengine/replay.go finalizeEntered and withoutFinalizeEntered: settle whether the Raddle-File exemption applies only to an entry origin ended as done, so an entry removed from origin's board while the pair stands at Raddle-File still reconciles as ended."
  - "internal/loomcli/wiring.go sketchFinish and sketchRefusedNotice: settle whether a failed notice or a failed draft write leaves Finish returning nil, whether a test row pins that, and whether the notice's first way forward is a runnable command with the payload filled in."
  - "Docs touched by the done-run clear and the Raddle-File exemption (DobbyTasks godoc in internal/battencli/dobbyreads.go, internal/dobbyengine/doc.go, internal/loomcli/doc.go, internal/fabricengine/doc.go): settle whether each states the exception that a done motif run of a raddle task carries no start refusal and that a raddle run at Raddle-File is not read as ended."
---

Focus targets the sites round 1 found open; the reviewer judges each from the diff alone.
