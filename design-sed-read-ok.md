# Agent shell hygiene: read-only sed allowed, no scripted edits or /tmp rm in Webster forks

`sed-read-ok` · Task · bug, enhancement, stencil, webster, shuttle · awaiting · PR-Gate

## Idea (operator, 2026-10-11)

Make `sed` work well for agents: reading with `sed` is allowed, and only rewriting files with a script stays banned.
The operator has updated `scribe:conversation` (1.1.0): its old "never use `sed`, even to read" rule is gone, and the remaining rule is to change files only with Edit or Write.

## Seen (2026-10-11)

- `contracts/stencils/edit/edit-directive.md` (rendered into every spawned role) bans `sed` only as a file-rewriting script; `contracts/stencils/stencils.go` `editRulePhrases` matches "`sed`" as one of the rule's phrases.
- `internal/websterengine` classifies a `sed -n` line or range print as a read, and any other `sed` form as a write.
- `webster-fork-shell`: a Webster fork used `sed -i` and `rm` on a computed `/tmp` path and stalled on a Claude Code permission prompt.
- The orch passed "never sed" into agent briefs from the old scribe rule.

## Open

- Whether any lyx-spawned role, hook or audit still refuses or prompts on a read-only `sed` (`sed -n`), and whether the Webster classifier should admit more read forms.
- Whether stencils, the orch role or `CLAUDE.md` carry a stale "never sed" wording.
- Whether a read-only `sed` allow rule belongs in the settings lyx gives spawned agents.

## Merged: `webster-fork-shell` (2026-10-11, bundled 2026-10-12)

Seen in `shuttle-hook-first`, Webster batch 01: the implementer fork ran, from Merriam's session:

```
tmux -L askexp kill-server; rm -rf "$(cat /tmp/claude-1000/askdir)" /tmp/claude-1000/askdir; cd internal/shuttlecli && sed -i '604s|Write(|Edit(|' smoke_turns_test.go && sed -n 603,605p smoke_turns_test.go
```

- Claude Code stopped it with a permission prompt ("Dangerous rm operation on statically-unresolvable target"), which fires in bypass mode too, so the run waited on the operator.
- The fork edited a test file with `sed -i`, against `contracts/stencils/edit/edit-directive.md`.
- It kept experiment state in `/tmp` and ran its own tmux server (`-L askexp`); no Webster stencil names a scratch place or forbids a system temp directory.

Open:

- Whether the edit directive reaches Webster implementer forks at all, and if so why the fork ignored it.
- Whether the implementer stencil should name a scratch place inside the pair and forbid `rm` on a computed path.
- Detection (operator, 2026-10-11: such a stall must be noticed, it likely sat a while): `shuttle-hook-first` D5 installs a `PermissionRequest` hook that reads the session as asking at once.
  Whether the hook fires for a fork's prompt in Merriam's session, and what turns a run that stays asking past a bound into a notice that reaches the operator.
