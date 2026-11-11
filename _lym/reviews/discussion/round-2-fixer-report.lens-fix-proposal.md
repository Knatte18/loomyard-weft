```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out as proposed (fix-proposal lens).

Proposals challenged: F1, F2, F3, F4, F5, F7 (the proposed part), F8, F9, F10, F11, F12. F6 and F13 were fixed with the fixer's own change, so they are not in scope.

**1. MEDIUM. F5 strands pre-change sessions that ran under the operator's shell `CLAUDE_CONFIG_DIR`.**
- Changed text: D4's new bullet ("`ScrubbedEnv`, also names `CLAUDE_CONFIG_DIR` … An operator's shell value … then reaches no lym pane either") and D6's new fallback ("the `env` module's value, then the default").
- Case: an operator with no `claude.accounts` and no `env` value has run lym with `CLAUDE_CONFIG_DIR=/x` exported in their shell. Today the tmux server inherits `/x`, so the orch's and dobby's transcripts sit under `/x/projects`.
- After the change, the server is scrubbed and the next resume runs under `~/.claude`. With an empty list, D2's first bullet says the launch "makes no pick, searches nothing and writes no record", so nothing redirects it, and the resume check refuses every such session.
- Neither the record nor its migration notes say this, and the D1 criterion that an empty-list launch "neither set nor unset" no longer means "stands as today" for this operator.
- Evidence: D2's first bullet; D4's new bullet; `internal/reedengine/lifecycle.go:456`, which spawns the server from `os.Environ()`.

**2. MEDIUM. F3's resume-check rule contradicts D2's empty-list bullet.**
- Changed text: D2's new bullet: "with no record it searches every known directory for the session's transcript … `Prepare` then records the directory the same search finds."
- The rule is unconditional. D2's first bullet says that with `claude.accounts` empty a launch "searches nothing and writes no record", and the D1 criterion says "a resume launch writes no record".
- So with an empty list and the transcript found under the `env` module's directory, the new bullet has `Prepare` write a record and the first bullet forbids it.
- Evidence: D2's first bullet; the D1 acceptance criterion; F10's own premise, which the fixer applied to the criteria but not to this new bullet.

**3. MEDIUM. F9's way forward fails for any worktree that has sessions.**
- Changed text: D2: "the way forward is to log the directory in again or remove the record so the next fresh session searches and picks."
- Case: a worktree recorded on `/acct-b`, which still exists but is logged out, with resumable sessions whose transcripts are in `/acct-b/projects`. The operator removes the record. The next fresh launch searches every known directory, and the existing `/acct-b` is among them, as one of the listed accounts or a record's directory.
- The search finds the transcripts there, records `/acct-b` again and makes no pick, so the worktree is back at the login prompt.
- The record already says this in its Auto-mode assumptions: "removal alone moves only a worktree with none". The working ways forward are to log in again or to write the record with another `config_dir`.
- Evidence: D2's fresh-launch bullet; D3 step 1; the Auto-mode assumptions.

**4. LOW. F4's "the caller" names no layer that holds the three id sources.**
- Changed text: D2: "The caller resolves those ids and passes them to `Prepare` and `OpenLaunch` as told data in the launch spec."
- The direct callers are shuttle's runner (`internal/shuttleengine/run.go:642`) and `internal/shuttleengine/open.go`. Neither holds reed's strand list or the orch's state record, and nothing in `shuttleengine` imports `orchengine`.
- So either every verb that starts a shuttle run collects all three sources itself, or shuttleengine gains a dependency on orch's record. The record names neither, so an implementer has to decide where the collection lives and which modules may read orch state.
- Evidence: the grep below; `Claude.Prepare(runDir, spec, cfg)` at `claudeengine.go:121`.

No case found against F1, F2, F7's proposed part, F8, F10, F11 or F12.

Tool calls:
- One grep of `orchengine`'s imports of shuttleengine, shuttle's imports of `orchengine`, and the `OpenLaunch` callers. My context did not show which layer calls `Prepare` and `OpenLaunch` or the import direction between shuttle and orch.
