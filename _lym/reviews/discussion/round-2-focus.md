---
round: 2
exclude_lenses: []
focus:
  - "decision-record.md D10 and D11: after the fixes, check that Beam-Await's recovery from a crash between the accept's removals and Done, and from a goto back from Finalize, names who lands the beam again and why that actor acts, given that the dobby keeps its land episode in its state file across restarts (`internal/dobbyengine/land.go` `recordLand`) and the accept removes the dobby-stop marker with the land request (`internal/pawlengine/beamawait.go` `accept`). Settle whether every re-step or re-land path states what bounds it."
  - "decision-record.md D2, D3 and D10 Bounds: check that every path that starts a member run under a beam whose ready record stands, the dobby's start hold and a hand `lym batten run` through the RunHolds check in `internal/battencli/beamfork.go`, is named with its refusal, and that each Bound cites a guard that exists."
  - "decision-record.md D3 refusal 6, D8 and D13: check that a recorded pull request merged on GitHub at a head other than the beam's head, a GitHub merge refusal Finalize meets after pushing (`classifyRefusal` in `internal/pullrequest/client.go`), and a pawl run left at Finalize by the old binary with a pull request only in its scratch record each end in a stated halt or carry, with a way forward that fits a beam."
  - "decision-record.md D13 and D4: check that every tracked file the move writes or removes (the landing record, the ready record, a reopen request) states who commits it, and that no file the move leaves behind has lost its only reader."
---

Round 1 raised its gating findings around D10, D11 and the ready window, so those are the areas to look at again.
No lens is excluded: the fixes add text that no lens has seen.
