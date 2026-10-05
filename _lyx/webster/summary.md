# agent-command-wrapping: env-file stdin, short step envelope, bare driver step

All three batches recorded `done`.

## 01-claude-env-file (head `1c46e61aa`)

The Bash stdin rewrite hook is replaced by a `CLAUDE_ENV_FILE` that `Prepare` writes and both pane launch lines export.
The rewrite hook's code, its settings entry and its integration and smoke tests are deleted; new tests cover the env file and a `settings.json` whose only Bash entries are the fork guard.
One deviation outside the declared file-ops: `internal/shuttlecli/modelpin_test.go`, where `wantSpawnSiteCount` dropped from 5 to 4 because deleting the smoke test removed a real-claude spawn site.

## 02-short-step-envelope (head `fb2f22542`)

`lyx shed step` writes the full envelope to its record first, then prints a short envelope carrying `envelope_path`.
`--full`, or a failed record write, prints the full envelope instead.
The `Verbs` comment, `doc.go`, the `loom step` help text, `PATTERN-shed-verb-set` and `docs/overview.md` were updated to match.
Tests: a table over success and refusal, each with and without `--full`, replaces the old record-equals-printed-line test.
No deviations.

## 03-driver-bare-step (head `264d74844`)

The driver stencil shows `lyx shed step {{.run_id}}` as the whole command and replaces the cd-subshell rule with a no-cwd-change rule.
Its envelope-key wording now uses `envelope_path`.
`TestDriverStencil_ShowsOneBareStepCall` pins the bare call.
No deviations.

## Notes

`record-batch 03` logged a `stencilstore` warning: the board copy of `shed-template-driver` is behind the worktree source until a production deploy.
The live `./deploy-dev` check from the plan's Shared Decisions was not run and is left for the PR.
