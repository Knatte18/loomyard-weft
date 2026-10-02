# Card 1 — spec-permission-mode

**Edit:**
- `internal/shuttleengine/spec.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/claudeengine.go`
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine/command_test.go`
- `internal/shuttleengine/claudeengine/prepare_test.go`

**Intent:**
Add `PermissionMode string` to `Spec`, documented as engine vocabulary exactly like `Effort`: `validate` never inspects it, and the engine is the sole validator.
Rewrite the `Interactive` field's doc so it no longer claims autonomous runs add `--dangerously-skip-permissions` by themselves; the flag now follows the resolved permission mode, and `Interactive` keeps governing the AskUserQuestion hooks.

In `claudeengine`, add a `validatePermissionMode(mode string, interactive bool) (skipPermissions bool, err error)` helper beside `validateEffort` in `command.go`:
empty resolves to the run mode's default (autonomous skips, interactive prompts), `bypass` skips, `prompt` adds nothing.
`prompt` on an autonomous run is an error, since an autonomous run that prompts stalls with nobody to answer, and any other value is an error naming the three legal values.
`Prepare` calls it before any artifact is written, next to `validateEffort`.
`buildLaunchCmd` and `buildResumeCmd` take the resolved `skipPermissions` bool in place of deriving the flag from `interactive`, so the launch line and the resume line reed replays always carry the same mode.
Update both functions' doc comments, which today tie the flag to `!interactive`.

Bound: this removes the permission-prompt guard only for a run whose caller sets `bypass`;
every existing caller leaves the field empty and keeps today's behaviour in both run modes.

Tests: an interactive run with `bypass` carries `--dangerously-skip-permissions` on both lines;
empty keeps today's lines for an interactive and an autonomous run;
`prompt` on an interactive run carries no flag;
`prompt` on an autonomous run and an unknown value fail `Prepare` before `prompt.md` or `settings.json` exists.
Add a short paragraph to the `internal/shuttleengine` package doc naming `PermissionMode` as caller-owned engine vocabulary.

**ImpactSummary:** Adds one Spec field and threads a resolved bool through the two claudeengine command builders; every existing caller leaves it empty and keeps its current flags.
