# Codex engine trial: home, trust, sandbox and refusals

Adds `docs/research/codex-engine-trial-home-sandbox-2026-11-03.md`, the report of seven cases run on real Codex sessions (`gpt-6-luna`, effort medium) launched through `lym shuttle run`.
No code changes.

## What the report finds

- Every case matches `internal/shuttleengine/codexengine/doc.go`: the lym Codex home and its links, the whole-rendered `config.toml` under the file lock, the dropped vanished root, the Warn on a regular `auth.json`, the `-a never -s workspace-write` launch line and its refusals, the `.codex/rules` refusal and `.codex/config.toml` Warn, and the trust-prompt backstop.
- The operator's `~/.codex/config.toml` has the same sha256 and mtime before and after.
- Case 6 shows the `.codex` check reads presence, not git tracking, and the rules refusal's way forward names an assist model key that a plain `lym shuttle run` has not.
- Case 7 rests on one log line (`dismissed startup gate`) that does not name the gate.

## Faults filed as board notes

- `shuttle-run-no-model-spec`: `lym shuttle run --model` takes no model spec and has no engine choice.
- `shuttle-open-refuses-codex`: `lym shuttle open` refuses the codex engine for lack of a prompt.
- `codex-test-writes-real-home`: `TestPrepare_AppendsStandingPassages` writes the real `~/.lym/codex`.

## Design decisions

- The cases ran with a local `engine=codex` override of shuttle.yaml in this worktree, set by the parent's direction because `run` has no engine choice; the override was removed afterwards and is not committed.
- Case 6 used untracked `.codex` files, since committing trial files to the task branch would land them; the report notes the difference from "committed".
- Case 2 launches two sessions in one worktree, so both share the same roots.
