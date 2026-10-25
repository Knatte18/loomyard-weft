# Card 7 — draft-writers

**Edit:**
- `internal/battencli#clearRunClaim`
- `internal/battencli#TestClearRunClaim`
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/pairteardown#Teardown.settleBoardClaim`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/battencli#seedBoardTask`
- `internal/battencli#seedBeamBoard`
- `internal/battencli#TestBeamLanding_ReachesWorktreeCreateSkip`
- `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver`
- `internal/battencli/cli.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/boardengine#StatusDraft`

**Intent:**
Make every lyx path that clears a task's status write `draft`, and refuse a fresh start of a draft (D5), so a null status reaches the board only from a payload the operator or the orch writes.
`clearRunClaim` writes `draft` in place of nil when it clears a run-form status, so `lyx batten cancel`, with or without `--remove`, and promote's claim clear leave the task `draft` and it is not started again.
`settleBoardClaim`'s `claimClear` arm writes `draft` through `writeBoardClaim` instead of a nil status, so a task whose pair `lyx fabric remove` removed while unseeded, paused, blocked, failed or awaiting is not started again; `claimDone` still writes `done`.
`startGuard` gains a `Draft` seam, filled by `hubStartGuard` from the local board, reporting whether the slug is a task whose status is `draft`; `refuse` refuses a `run` or `step` with no status file of such a task with "`<slug>` is a draft task, which no run starts; way forward: run `lyx board set-status '{"slug":"<slug>","status":null}'` to release it, then `lyx batten run <slug>`", the slug filled in so the way forward runs as typed.
The input that reaches the refusal after every earlier guard passes is a fresh `lyx batten run` of a task created by `lyx board upsert` and never released; a resume (a status file present) is not refused, since its entry holds a run status.
The cancel paragraph of `cli.go`'s package comment and the batten cancel sentence of `docs/overview.md` say the claim is left `draft`.
The tier1 fixtures that create a task and then start it with a fresh `lyx batten run` release it right after the create, with `SetStatus(slug, nil)`, so the new refusal does not stop them: `seedBoardTask`, `seedBeamBoard`, the board seeding inside `TestBeamLanding_ReachesWorktreeCreateSkip` and inside `TestAdoptIntegration_SecondCloneTakesOver`.
Tests: `TestClearRunClaim` asserts the write is `draft`; `TestTeardown_CallSequence`'s clear rows expect `write=draft`; `TestStartGuard_Refuse` gains a draft row whose error equals the refusal text with the slug filled in, and a released row (nil status) that passes.

**ImpactSummary:** Two board writers change their written value and the start guard gains one refusal; four tier1 fixtures release the tasks they start, and the claim settlement's other arms and the cancel order are unchanged.
