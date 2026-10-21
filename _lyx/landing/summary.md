# Intake reads the hub's own repository; orch stops self-running intake

`lyx board intake list|import|close` now works against the repository the hub works on, not the hard-coded `Knatte18/loomyard`.
A quarry hub's intake reads `Knatte18/quarry`; only the loomyard hub's intake reads `Knatte18/loomyard`.
Before, a quarry orch running its role's wake-time intake imported LoomYard issues onto the quarry board.
Selfreport stays the LoomYard-only bug filer, and the orch role no longer runs intake on its own or reviews escalated requests.
No issue is closed by this change.

## What changes

- New package `internal/hubissues`: a `Repo` value bound to one repository, with the four intake calls (list open issues, get one, list comments, comment and close).
  `Open(remoteURL)` parses the URL with `githubclient.ParseOwnerRepo` and builds the client with `githubclient.New`.
  Intake no longer touches `internal/selfreportengine`.
- `internal/selfreportengine` loses `inbox.go` and its tests, and keeps `CreateIssue`, still filing to `Knatte18/loomyard`.
  Its docs now say it files LoomYard issues only.
- `fabricengine.HubRepoURL` returns the hub's warp binding when one is recorded, else the `origin` URL of the location's warp worktree.
  A hub with neither gets an error naming both sources; a URL that is not a GitHub URL is refused with a message naming it.
- `internal/boardcli` resolves the repository only when an intake verb runs, so every other board verb stays offline.
  An intake verb with no resolved location (the `--board-path` sync child) is refused.
  Intake output keeps its shape.
- Orch role stencil (`contracts/stencils/orch/orch-template-role.md`):
  - no inbox check at wakes or on `landed`;
  - intake runs only when the operator asks, and its fork procedure stays;
  - a finding in another repository is filed to that repository's own issues;
  - on a parent-review or PR-Gate escalation, and on the arbiter and peer hand-on forks, the orch puts the alignment agent's open question to the operator and submits the answer; it never reviews and never forks a review.
- Docs follow: `docs/overview.md`, `internal/boardcli`, `internal/boardengine` and `internal/orchcli` docs and comments, the `ly:board` skill, and the new package's `doc.go`.

## Design decisions

- The binding wins over origin when both exist and differ, since it is the hub's recorded pairing.
  The origin read lives in `fabricengine`, the one package that runs git on warp.
- A hub whose warp URL is a local path or a non-GitHub host can no longer run intake; it used to silently read `Knatte18/loomyard`.
- A `Repo` value replaces the package-level client seam, so the moved tests run in parallel.
  They keep the old coverage: pagination, pull request exclusion, comment order, comment-then-close, a failed close naming the posted comment, and an unresolvable token.
- The `InboxIssue` identifier in `internal/boardengine` is unchanged; only its comments changed.

## To do by hand

- After this lands and deploys, remove the "Intake only on the operator's word" and "No parent reviews" override sections from the hub-wide orch rules with `lyx orch rules`.
  That file lives on Bolt, so no task commit can carry the removal, and removing it before the new role is deployed reinstates the old behaviour.
  Until then the sections restate the new role and do not conflict with it.
- Run `lyx board intake list` in a quarry hub and a loomyard hub and confirm each lists its own repository's issues; the tests use stubbed clients, so no real GitHub call was made.
- Check the stale-documentation fix for intake in `internal/boardengine/doc.go:96` and the comments in `internal/boardengine/intake.go` and `internal/boardengine/task.go` reads correctly.
- Check that the `hubRepo` closure in `internal/boardcli/cli.go:48` is covered: the `--board-path` refusal and the no-binding-no-origin refusal go through `Command()` in `internal/boardcli/cli_test.go`, and no test pins that a non-intake verb never resolves the repository beyond the existing `list --board-path` subtest.
- Check that the import and close handlers' unresolved-repository refusal (the `hubRepo()` error branch) leaves `board.json` unchanged, covered by rows in `internal/boardcli/intake_test.go`.
- Check that the neither-source error asserted in `internal/fabricengine/hubrepourl_integration_test.go:44` names both the binding file and `origin`.
- Check that the friction-findings sentence in `internal/orchcli/doc.go:47` reads correctly now that intake runs only on the operator's word.
