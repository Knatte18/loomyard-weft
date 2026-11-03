# Merge origin/main into the codex-engine beam

Brings the `codex-engine` beam up to date with `origin/main`, so the beam's landing merges into main cleanly.
The merge applied without conflicts and without any follow-up edit: the diff against the beam is main's own changes, and the beam's Codex work (assistant reviewer seat, Codex skill loading and shell guard rows) is untouched.
No new behaviour is added.

## What main brings in

- `loomcli`: a pre-push check of origin and owners before a run commits or pushes (`prepush.go`), review-status and checklist changes, and a `loom status --watch` that stops printing a line per poll while a wait runs.
- `hibernatecli`, `hibernateengine` and the new `hibernateconfig`: the pause lifts on wake only when a switch allows it.
- `fabricengine`: `remove` deletes worktree dirs that were refilled with ignored files; failed-push and origin-reconcile handling changed.
- `battencli`: a stray dir reads as a gone pair, a beam landing keeps its mark, and cancel and cleanup were reworked.
- `beamcli`: `create` clears a done motif run under the same slug.
- `dobbyengine`: finishes a paused run whose work already reached its parent.
- `gatecli`: ends a tier whose shell exits while an orphan holds its pipe.
- `shedrun`, `shedverbs`, `tendercli` and `adoptcli`: child-recipe lookup, status output and smaller follow-ons.
- Way-forward texts name a `lym board remove` command the verb accepts; `PATTERN.md`, `pattern/` and `docs/overview.md` carry the matching doc updates.

## What to check

- `git diff --cc ca9b3917` is empty, which confirms no hand-resolved hunk; a reviewer can skip line-by-line review of main's side, already reviewed on main.
- The merged tree was not built or gated in this description step.
  Run `lym gate build` and `lym gate test` on the branch before landing, and in particular exercise `beamcli`, `battencli`, `loomcli` and `gatecli`, where main's changes sit closest to the beam's new Codex code.
