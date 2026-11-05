# Relevant code

- `contracts/orchrole/orch-role.md` — the orch role that gains the Slack mail and mirroring rules
- `internal/creelengine/doc.go` — creel's sender identity, program mailboxes, receivers, caps and retention the relay builds on
- `internal/loomshed/operatorwait.go` `OperatorWaitRows`, `WaitsOnOperator` — the operator-wait rows the reminders read
- `internal/orchcli/resumecontext.go` `resumeContextCommand` — the resume pointer that carries the presence line
- `internal/githubclient/cache.go` `cacheDir` — the machine-global credential directory the Slack token file joins
- `internal/orchcli/start.go` `BeamUp` — the prime bring-up that starts the relay
- `internal/runproc/record.go` `HubOwner` — the owner the relay process registers under
