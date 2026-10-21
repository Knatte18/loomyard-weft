# Intake reads the hub's own repo; orch stops running intake and reviewing

`lyx board intake` used to list, read, comment on and close the issues of the selfreport repository, hard-coded to `Knatte18/loomyard`, whatever hub it ran in.
The quarry hub's orch imported LoomYard issues onto the quarry board that way.
This change gives intake the hub's own repository, keeps selfreport LoomYard-only, and takes intake and reviewing out of the orch's own initiative.

## Intake

- `fabricengine.HubRepoURL(l)` returns the repository the hub works on: the `.lyx-warp` binding at the hub's board dir when recorded, else the `origin` remote of the location's warp worktree, else an error naming both sources.
- The new package `internal/hubissues` holds intake's GitHub side.
  A `Repo` is bound to one repository: `Open(remoteURL)` parses it with `githubclient.ParseOwnerRepo` and authenticates through `githubclient.New`, refusing a non-GitHub URL by name.
  Its methods are the four calls intake makes: `ListOpenIssues`, `GetIssue`, `ListIssueComments` and `CommentAndClose`, with the same pagination, per-request deadline and error classification the old calls had; an API rejection now names the repository.
- `internal/selfreportengine` loses `inbox.go` and its tests and keeps only `CreateIssue`, still filing to `Knatte18/loomyard`.
  Its package doc says it files LoomYard bugs only, whatever hub lyx runs in.
- `internal/boardcli` keeps the resolved location from its pre-run and opens the hub repository only when an intake verb runs, so every other board verb stays offline.
  The handlers take the `*hubissues.Repo`, and an unresolvable repository is an error envelope.
  The `--help` texts, `doc.go`, `docs/overview.md` and the `ly:board` skill describe intake as reading the hub's own repository, and the skill says intake runs on the operator's word.

## The orch role

`contracts/stencils/orch/orch-template-role.md`:

- The inbox check is gone from every wake and the `landed` notice.
  A new `## Intake` section holds the intake procedure, run only when the operator asks, with the intake fork rules unchanged.
- Where a finding goes: a lyx bug still goes through `lyx selfreport create`; a bug in any other repository goes to that repository's own issues, never through selfreport.
- Parent-review, PR-Gate, the arbiter hand-on bullet and the peer hand-on bullet: the orch reads only the open question (the brief's "Why it did not settle the review" line, the reason in the PR-Gate notice, or the envelope's `question`), puts it to the operator in plain words and submits or records the operator's answer.
  It never reviews and never forks a review.
- Acting without asking no longer lists approving, and the questions the alignment agent or the arbiter hands on are among what the orch asks the operator.

## Design decisions

- Intake's calls got a package of their own rather than a repository parameter on selfreport's, since the board entry says intake never goes through selfreport.
  A repository-bound `Repo` replaces the package-level client seam, so the intake tests in `hubissues` and `boardcli` now run in parallel.
- The binding wins over origin when both exist, since it is the hub's recorded pairing; origin covers a hub that predates the binding.
- The hub-wide orch rules' "Intake only on the operator's word" and "No parent reviews" override sections are not removed by this change, although the board entry asks for it.
  That file lives on Bolt and is written only through `lyx orch rules`, so no task-branch commit carries it, and removing it before this role deploys would reinstate the old behaviour.
  The parent decided by message that the orch removes the two sections with `lyx orch rules` after this task lands and deploys; it could not record that with `lyx loom decision add`, since the decision record did not exist yet, so this is an unrecorded parent correction.
- The parent-review brief and the Bouncer escalation brief still address a one-shot fork.
  The role now says the orch reads one line or the envelope's question from them, so they were left as they are.

## Testing

- `internal/hubissues` carries the old inbox tests over to `Repo`, plus a refusal of a non-GitHub URL and a token-not-resolvable check through a failing transport.
- `internal/boardcli`'s intake tests drive a fake hub repository at `owner/hub-repo`, and the list table gains a row for an unresolvable repository.
- `internal/fabricengine`'s new integration test walks a copied hub from binding to origin to error.
- `lyx gate test --tags integration` passes over `boardcli`, `hubissues`, `selfreportengine`, `selfreportcli`, `fabricengine`, `orchengine`, `orchcli` and `cmd/lyx`.
