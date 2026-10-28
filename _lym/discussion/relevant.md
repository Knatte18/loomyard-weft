# Relevant files

- `internal/creelcli/hookread.go` `hookReadCmd`, `hookRead` — the claim verb D1 to D3 change and D4's waiter verb sits beside
- `internal/creelengine/doc.go` — the delivery, notice, reminder and interrupt rules D4 to D8 rewrite
- `internal/creelengine/reminder.go` `MessageLine`, `isNotice`, `composeReminder`, `reminderDue` — the line D8 quotes and the reminder machinery D6 removes
- `internal/creelengine/verbs.go` `Store.Interrupt` — the interrupt flow D7 runs before the post
- `internal/creelengine/seam.go` `ErrNoRunningTurn` — the outcome D7 still posts on
- `internal/creelcli/cli.go` — the `post` and `inject` flags, where `--interrupt` and the help text of D7 and D10 go
- `internal/shuttleengine/claudeengine/settings.go` `buildSettings`, `recordingCommand`, `creelMailHookCommand`, `hookContext` — renders the hooks D1, D2 and D4 change
- `internal/shuttleengine/claudeengine/settings.go` `steerTextForbiddenChars`, `noticeTextForbiddenChars` — the notice guard D9's sentence gets its own check beside
- `internal/shell/posix.go` `Quote` — single-quoting that carries D9's `"` literally; `internal/shell/pwsh.go` holds the pwsh form
- `internal/shuttleengine/claudeengine/events.go` `ParseEvents` — reads every Stop line as a turn end, the reason D2 merges the hooks
- `internal/shuttleengine/claudeengine/command.go` `buildLaunchCmd` — carries `--append-system-prompt`, where D9's sentence joins the deny notice
- `internal/shuttleengine/claudeengine/doc.go` — engine doc D11 updates
- `contracts/stencils/orch/orch-template-role.md` — describes the reminder and `lym creel read` turn D6 removes
- `internal/orchcli/doc.go` — describes batten notices arriving as typed lines or reminders
- `docs/research/creel-hook-channels-2026-10-28/report.md` — which hook channels deliver and show
- `docs/research/creel-midturn-trust-2026-10-28/report.md` — the block-plus-sentence result D1 and D9 rest on
