# Relevant files

- `internal/orchcli/start.go` `startCmd`, `adoptStrand` — the start verb, its adopt branch and guards that `--continue` reuses
- `internal/orchcli/stop.go` `stopCmd` — stop removes the strand, so the record must outlive it
- `internal/orchcli/session.go` `runnerSession` — orch's adapter over shuttle's reading the watcher uses
- `internal/orchcli/doc.go` — package doc naming `--adopt`; gains `--continue`
- `internal/orchengine/state.go` `State`, `ResetForFreshLaunch`, `saveStateForStrand` — the persisted record that gains the session id
- `internal/orchengine/watcher.go` — the tick that reads events and the session reading, where the id is kept current
- `internal/orchengine/prompt.go` `RenderAdoptPrompt` — the launch pointer the continue path reuses
- `internal/orchengine/launch.go` `SourceAdopt` — the prompt source the envelope reports
- `internal/orchengine/doc.go` — recovery and adopt sections plus the reed-rebirth residual
- `internal/shuttleengine/sessionread.go` `RunSessionState`, `readSessionFacts` — engine-neutral reading that already picks the newest signal's session id
- `internal/shuttleengine/sessionsignal.go` `SessionSignal` — the signal's engine-neutral `SessionID`
- `internal/shuttleengine/engine.go` `SessionResumer` — the resume check every resume goes through
- `internal/shuttleengine/claudeengine/resume.go` `CheckResume` — refuses a missing transcript or a live holder
- `cmd/lyx/testdata/test-budget.yaml` — orch package test budgets
