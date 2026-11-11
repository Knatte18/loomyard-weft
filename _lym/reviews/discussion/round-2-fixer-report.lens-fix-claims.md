```yaml
lens: fix-claims
outcome: reported
```

Task: check the claims the fix's added or rewritten sentences make against the code at HEAD.

**MEDIUM**: the D6 liveness sentence says something the code does not hold.
- **Changed text:** D6, "The engine behind the id-only liveness read is built with its location's `Dirs` the same way."
- **Summary:** Some engine sets are built from `Dirs` that carry no `Board`. Those load no machine layer, so they get no `claude.accounts`, and their liveness read misses sessions on a listed account.
- **Evidence:**
  - `internal/burlercli/wiring.go:202` calls `shuttleengines.New(shuttleCfg, configengine.Dirs{Anchor: res.StateDir})`.
  - `internal/configengine/doc.go:59` says "The machine layer applies only when `Dirs` carries both `Anchor` and `Board`."
- **Effect:** The sentence states a guarantee that call site does not provide. The record should either name the hub `Dirs` as a requirement on every engine set that answers liveness, or bound the claim.

**LOW**: one D5 claim borrowed from the reviewer is unverified.
- **Changed text:** D5, "runs `cd "${CLAUDE_PROJECT_DIR:-.}"` before the verb, as the creel hook commands do, so the verb resolves its hub from the worktree's anchor."
- **Summary:** It is not established that a `statusLine` command gets `CLAUDE_PROJECT_DIR` in its environment.
- **Evidence:** In `internal/shuttleengine/claudeengine/settings.go`, the code shows the variable only in hook commands (lines 117, 160, 179 and 207). The record's own facts sections say nothing about the status line's environment. Without the variable, the `:-.` fallback leaves the verb in the command's working directory, so "resolves its hub from the worktree's anchor" rests on that directory being inside the worktree.

**Claims that hold at HEAD:**
- `selfexec.LymBinary` exists (`internal/selfexec/binary.go:19`).
- Shuttle runs the resume check before `Prepare` (`run.go:581` and `run.go:642`).
- `CheckResume` is told the workdir, so it can resolve the worktree's record (`run.go:969`).
- The session-end recorder calls `LocateSessions(end.Worktree)` (`shedadapters/sessionusage.go:107`).
- `reportengine/cost.go` prices runs from the usage records in each run's tree.
- `PATTERN-cwd-resolution` has a "Told geometry" section.
- Reed's server spawn applies the engine scrub to `os.Environ()` (`reedengine/lifecycle.go:456`).

Tool calls:
- One Bash call to grep `PATTERN-cwd-resolution` and "Told geometry". My context had only the reviewer's citation of the pattern.
- The same call listed the `shuttleengines.New` call sites. My context did not show which `Dirs` the liveness-serving engines are built with.
- The same call read `internal/selfexec/binary.go`. My context had only the grep hit for `LymBinary`, not its contract.
