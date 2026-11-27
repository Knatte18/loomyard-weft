# Relevant files

- `docs/research/autonomy/stop-classes-2026-11-27.md` — the stop classes, counts and proposed table the design rests on
- `contracts/orchrole/role-autonomy.md` — the orch's autonomy rules and severe-halt classes the levels change
- `internal/heddleengine/glossary.md` — the autonomy and severe halt lines the design changes
- `internal/dobbyengine/template.yaml` — the `beam.autonomy` key and its comment
- `internal/hubgeom/autonomy.go` `Autonomy` — the one reader of a run's autonomy, a boolean today
- `internal/dobbyengine/waitreminder.go` `WaitReminderDeps`, `OperatorWaitState` — dobby's Slack reminders of a run waiting on the operator
- `internal/sheddriver/driverules.go` `decideDrive` — the driver's rule table the new rows join
- `internal/loomshed/operatorwait.go` `OperatorWaitRows` — the rows the quiet notice reads as waiting on the operator
- `internal/arbiterstop/arbiterstop.go` `Eligibility` — the arbiter's admission rule that refuses cause budget
- `internal/shedrun/paths.go` `StartNotParkedKind` — the driver_not_parked refusal the resume wait replaces
- `internal/creelengine/verbs.go` `Store.AwaitsReply` — whether a strand awaits a creel reply
- `internal/slackengine/outbound.go` — routes a program's post to the hub orch's channel and drops mail of an orch whose button is off
- `internal/orchcli/notify.go` `PostAsProgram`, `WithRunAutonomy` — posts program mail and stamps autonomy on halt notices
