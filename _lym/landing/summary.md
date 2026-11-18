# gate: run with no tier0, skip inert-only Finalize, name a test route

A fresh hub with no declared tiers now runs and lands tasks, a change of inert files alone runs no tier at Finalize,
and the darn writer has a permitted route to measure one test class.

Closes #135
Closes #133

## What changes

- `gateslot`: `Config.Gate` no longer refuses a hub that declares no tier at all, as the template does with `tiers: {}`.
  It returns the key with no tiers, so shed build renders the gate calls of a fresh hub.
  A hub that declares tiers but no `tier0`, and a key naming an undeclared tier, are still refused.
- `gatecli`: a `--gate` call in a hub with no tier prints a line on stderr saying no tier runs, then runs only the key's lints.
  Failing lints still fail the call.
- `gatecli`: a `--gate` key of scope `all` now checks the diff from the merge base with the parent branch.
  When every changed file matches `inert_paths`, the scope is empty and no tier runs, so Finalize no longer runs `dotnet test .` on a docs-only branch.
  An explicit `--all`, a worktree with no parent branch and an unreadable merge base stay whole.
  A tier with `carry: false` still runs whole.
- Darn writer prompt (gated Step 4 fragment): the writer may make one measuring run of a single test class through `lym gate test --gate darn-write`.
  It never runs a raw test command, since a raw run can exit 0 with no output when no test ran.
  When the gate's report says a project ran no test, the prompt names the `IsTestProject` property and a rerun with `-p:<property>=true` after `--`.
- Docs: the `gateslot` and `gatecli` package docs, `docs/dotnet-hub.md`, the `gate.yaml` template comments and the `gate test` help text describe both behaviours.

## Design decisions

- Rendering a key with no tiers, instead of passing a key that names an undeclared tier, keeps the refusal for a half-configured hub.
  The no-tier case applies only when `tiers` is empty.
- The silent exit 0 of #135 is handled by routing the writer through the gate, whose report says when no test ran.
  The guard around a raw `dotnet test` is not changed.
- The scope-`all` inert check applies only to `--gate` calls, so a hand-run `--all` still means the whole tree.

## To check by hand

Not run: a darn task with profile `docs` in a real hub with no tests and `inert_paths` set,
through shed build to Finalize.
The behaviour is covered by the integration tests added in `internal/gatecli`.
