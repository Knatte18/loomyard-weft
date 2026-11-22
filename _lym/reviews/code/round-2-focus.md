---
round: 2
focus:
  - "internal/fabriccli/roleguard.go `inOwnSandboxRoot`: in a task worktree `.lym` is a directory link to `<slug>-weft/.lym` in the records sibling. Does a verb run from, or a clone aimed into, `<worktree>/.lym/harden-sandbox/<hub>` pass for the strand of that slug, and is a target written in its resolved `-weft` form mapped back to its owner?"
  - "internal/fabriccli/roleguard.go `inOwnSandboxRoot`: for a target that does not exist yet, are symlinks in its existing ancestors resolved before comparison, so a link inside the sandbox pointing outside is refused and a symlinked worktree path is accepted?"
  - "internal/fabriccli/roleguard.go `inOwnSandboxRoot`: is ownership tied to the strand's own hub, so a same-slug worktree in another hub's sandbox root is refused?"
  - "internal/fabriccli/roleguard_test.go `TestSandboxedRoleGuard`: does the fixture wire `.lym` as fabric does, and does each guard behaviour (relative clone `--into`, the worktree-path check, link resolution, path-flag refusal ahead of the sandbox pass) have a row with a literal expected outcome that fails when it breaks?"
  - "internal/fabriccli/fabric.go header and internal/fabriccli/doc.go Merge section: do both describe the guard as the code now behaves, refusing a task session's mutating verbs outside its own sandbox root?"
---

Round 2 re-checks the sandbox-root ownership judgment against the production `.lym` link layout, then the test rows and the two prose sites.
