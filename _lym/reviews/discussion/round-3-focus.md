---
round: 3
focus:
  - "D5 with batten's Land row: after a local landing on the prime's branch, Land pulls the prime (internal/battencli/land.go landDeps, Fabric.Pull; internal/fabricengine/pull.go fetch and refuseUnsafeRewrite; internal/battenshed/land.go 'pull failed' Stuck). Settle what Land does while the parent push is owed, with origin reachable and with origin unreachable, and whether Scope and an acceptance criterion cover it."
  - "D5 and D3 with two runs landed on one parent while the first parent push is still owed: settle what state holds each run's landed tip, held records commit and pending done write, in which order they are released, and when the parent target's ledger entry and the all-clear go."
  - "D5's records hold and D13: settle what stops the catch-up's PushRun (internal/battencli/catchup.go), the pull-request record commit (internal/loomcli/landingdeps.go), cancel's records push (internal/battencli/cancelcmd.go), Worktree-Teardown, the pair teardown composite and batten's records in the prime (internal/battencli/commitstatus.go, internal/pairteardown/runread.go) from showing origin a done or landed run before origin's parent holds its work."
  - "D5 and the PATTERN-owner-machine constraint: settle which owner verdict the parent target's pusher reads before each parent push and done write, for the prime's landing target and for a beam parent, what it does when another machine owns the parent, and how its run after an absence is ordered behind the catch-up."
  - "D3's refused mark and owed entries: settle which writer removes each after a hand `lym fabric push` clears the refused-push record (internal/fabricengine RefusedPushStands, pushclassified.go; internal/runpusher start.go and run.go) or after any push outside the pusher makes the target level, and whether the all-clear then posts."
  - "D9's removal base: internal/loomengine or landing code writes the merge record in mergeParent (finalize.go) only after the after-the-merge-in clean check, and parent-align (parentalign.go) also writes one. Settle where the base comes from on a first merge-in, on a later merge-in and on a resume after a kill between the merge-in and the record write, and which red-first criterion covers it."
---

Each entry is an area where the design is still open after round 2.
