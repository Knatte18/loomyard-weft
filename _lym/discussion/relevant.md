# Relevant files

- `internal/shuttleengine/claudeengine/claudeengine.go` `Claude.Prepare` — builds the session's launch and resume commands and writes its settings
- `internal/shuttleengine/claudeengine/scope.go` — the only reader of CLAUDE_CONFIG_DIR today, for the installed-plugins lookup
- `internal/shuttleengine/claudeengine/transcriptread.go` — locates transcripts under the home directory's .claude/projects
- `internal/shuttleengine/claudeengine/audit.go` — resolves the Claude home directory for transcript lookups
- `internal/shuttleengine/claudeengine/prober.go` — reads the session registry under the home directory's .claude/sessions for liveness
- `internal/shuttleengine/claudeengine/resume.go` — reads the session registry and transcript for the resume check
- `internal/shuttleengine/claudeengine/usagelimit.go` — parses the usage-limit reset time
- `internal/shuttleengine/claudeengine/config.go` — the claude config module whose spec classifies each key
- `internal/reedengine/panebin.go` — composes the pane launch line, env module exports ahead of the launch command
- `internal/hubgeom/hubgeom.go` — the pane-env closure that hands the env module's variables to reed
- `internal/envconfig/config.go` — the env module and its refusal of reserved names
- `internal/orchengine/usagelimit.go` — the orch watcher's wait on a usage limit
- `internal/reportengine/cost.go` — the cost report reading transcripts through the engines' readers
