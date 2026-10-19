# A rejected weft push goes unnoticed, and a diverged weft is never repaired

`weft-push-failure-silent` · Note · bug, fabric, creel, batten

Imported from [issue #737](https://github.com/Knatte18/loomyard/issues/737).

## What happened

On the operator's work PC the GitHub account in use had read-only access to the records repo.
Every detached weft and board push failed with a 403 from 2026-10-16 to 2026-10-19, and lyx said nothing outside a WARN in the detached child's trace log under `_board/.lyx/logs/`.
The prime's `main-weft` fell 552 commits behind origin unnoticed.

## Gaps

- **A rejected detached push is silent.**
  `lyx cleanup --apply` reported `push_error: ""` because the push was spawned, not because it landed.
  A push that fails for a non-transient reason (auth, permission, non-fast-forward after the recovery rebase) should reach the operator: a notice to the orch through creel, or a status the next `lyx cleanup`, `lyx fabric status` or `lyx board` call prints.
- **`lyx cleanup` judges runs from a stale weft.**
  It never fetches first, so it listed three runs as paused that origin had deleted days earlier, and its removals diverged the weft from origin.
  It should pull the weft first, or refuse while the weft is behind its upstream.
- **`lyx fabric pull` leaves a diverged weft alone and still reports `ok: true`.**
  `git pull --ff-only` fails, the pull logs a WARN and returns `weft_pulled: false` inside an ok result.
  The doc names `git -C <weft> reset --hard origin/<branch>` as the manual recovery.
  Instead, pull should rebase the local weft commits onto the upstream and drop those that become empty, stopping with a named way forward only on a real conflict, such as a `status.json` both sides changed.
