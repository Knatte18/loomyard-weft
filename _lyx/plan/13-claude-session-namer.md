# Card 13 — claude-session-namer

**Create:**
- `internal/shuttleengine/claudeengine/sessionname.go`
- `internal/shuttleengine/claudeengine/sessionname_test.go`
- `internal/cliwire/sessionnamer.go`

**Edit:**
- `internal/shuttleengine/claudeengine/audit.go`
- `internal/cliwire/doc.go`
- `internal/cliwire/seam_enforcement_test.go`
- `internal/reedcli/watchdog.go`

**Uses:**
- `internal/reedengine/namerepair.go`
- `internal/shuttleengine/claudeengine/session.go`

**Intent:**
Fill reed's `SessionNamer` seam with the Claude implementation, wired by `cliwire`, so the watchdog repairs a Claude session name that a compact reset.

`sessionname.go` gives `*claudeengine.Claude` the three seam methods, with a compile-time `var _ reedengine.SessionNamer = (*Claude)(nil)`:
- `SessionNameDrift(sessionID, workdir, want)` reads the transcript at `claudeProjectDirFor(workdir)/<sessionID>.jsonl` through the streaming `forEachTranscriptLine` and takes the latest line whose `type` is `custom-title`, reading its `customTitle` field (`transcriptLine` in `audit.go` gains that field).
  It reports drift when that title differs from want, and when the transcript has no such line at all (Claude never got the name, or a compact dropped it).
  A missing or unreadable transcript logs `logger.Warn("claudeengine: could not read the session name", …)` with the path and cause and reports no drift, so a session whose transcript is not yet written is never retyped.
- `SessionIdle(capture)` delegates to the existing `IdleSession`, the same probe the orch watcher gates `/clear` on.
- `RenameText(want)` returns `/rename ` + want.
The file doc records the risk: `custom-title` is a Claude Code internal, and if its shape changes every read finds no entry, so the watchdog retypes `/rename` on each idle tick and logs each one.

`cliwire/sessionnamer.go` exports `SessionNamer() reedengine.SessionNamer`, returning `claudeengine.New()`, the one place a provider is chosen for reed.
`cliwire/doc.go`'s fixed dependency set and `seam_enforcement_test.go`'s `cliwireAllowedImports` gain `internal/reedengine` and `internal/shuttleengine/claudeengine`, with the doc saying why: reed is provider-blind and the watchdog daemon needs a provider filled in by the CLI wiring layer.
`reedcli`'s `enterSession` passes `cliwire.SessionNamer()` to `WatchNames` in place of nil.

`sessionname_test.go`, with `HOME` pointed at a temp dir by `t.Setenv` and transcripts written under it: a latest `custom-title` differing from the name reports drift, an earlier matching one superseded by a later different one reports drift, a matching latest one reports none, a transcript with no `custom-title` line reports drift, and a missing or unreadable transcript reports none and logs the could-not-read line (captured through `logger.SetOutput`);
`RenameText` returns the exact `/rename` text.

**ImpactSummary:** Widens cliwire's import allowlist by two packages and gives the reed watchdog daemon a real provider; no other cliwire caller changes.

**Commit:** 13: claude engine: repair a reset session name from the transcript
