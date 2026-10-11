# Agents may read with sed; only script rewrites stay banned

`sed-read-ok` · Note · enhancement, stencil, webster, shuttle

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
