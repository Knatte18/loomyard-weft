# Card 6 — claude-resume-check

**Create:**
- `internal/shuttleengine/claudeengine/resume.go`
- `internal/shuttleengine/claudeengine/resume_test.go`

**Uses:**
- `plan:internal/shuttleengine#SessionResumer`
- `plan:internal/shuttleengine/claudeengine#validateSessionID`
- `internal/shuttleengine/claudeengine#claudeProjectDirFor`
- `internal/shuttleengine/claudeengine#claudeHomeDir`

**Intent:**
Implement `shuttleengine.SessionResumer` on `Claude` in a new `resume.go`, with a compile-time assertion beside the method.
`Claude.CheckResume(sessionID, workdir)` derives the project directory with `claudeProjectDirFor(workdir)` and the session registry directory as `<claudeHomeDir()>/.claude/sessions`, then delegates to a pure `checkResumable(sessionID, projectDir, registryDir string, alive func(pid int) bool) (warning string, err error)`, which production calls with `proc.IsAlive`.
Every registry and transcript-layout fact stays in this file, per the Shuttle Provider-Seam Invariant, and both directories come from the paths the fork audit already derives, per the Told-Geometry Invariant.

`checkResumable` checks, in order:

1. `validateSessionID` passes; otherwise refuse, naming the expected id shape and that Claude's `/status` shows it.
2. `<projectDir>/<sessionID>.jsonl` exists; otherwise refuse, naming the directory searched and that only a session run from this directory can be resumed.
3. No registry entry holds the id while alive.
   Each `<registryDir>/*.json` entry decodes `pid` and `sessionId`;
   an entry whose `sessionId` matches and whose `pid` `alive` reports alive refuses, naming the pid and telling the operator to exit that session first.
   A matching entry with a dead pid is stale and proceeds.

A registry directory that is absent or unreadable, or an entry that cannot be read or decoded, returns a warning naming the path and that a live holder could not be ruled out, and the check proceeds, since only a positively identified live holder refuses.
Bound: an unconfirmed holder gets through only with that warning on the log and envelope, and the transcript check still confines a resume to a session recorded under the pane's own cwd.
Nothing here kills, signals or edits any process or file.

Tests in `resume_test.go` over a temp project directory, a fixture registry and a fake `alive`:
a malformed id refuses before any file is read;
a missing transcript refuses;
a live matching pid refuses and names the pid;
a dead matching pid proceeds with no warning;
a non-matching live entry proceeds;
an absent registry directory proceeds with a warning;
an undecodable entry proceeds with a warning.
