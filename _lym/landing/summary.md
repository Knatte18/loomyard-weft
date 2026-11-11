# Refresh the README: shorter, no dead Autoboard link, shorter Lineage

The README had drifted from the code, carried long prose blocks, and linked a removed repository.
This change rewrites `README.md` in one pass and touches no other file.

## What changed

- Lineage names Autoboard and Willie Tran as plain text with a note that the repository was removed.
  The section is cut to the Millhouse origin and the credits, and says LoomYard has grown beyond it.
- The run's seven steps are one table, and the four user-facing recipes are a second table.
  The recipes table replaces the `Beams` and `The board` recipe list that sat in the hub section.
- The `Under the hood` section is gone: the hub and worktree geometry, board entry kinds, beams and the `lym` command tree.
  `docs/overview.md` and `docs/getting-started.md` hold that detail, and the README's reading list points to them.
  The names table stays and now notes that dobby and orch are one per beam.
- The line announcing a planned Slack integration for `creel` is removed, as the Slack link between machines does not exist yet.
  The README will name Slack for two machines once it does.
- The measurements table, the open-data list and the reading list keep their links and are trimmed to what a reader needs first.

## Design decisions

- Detail moved out is not duplicated into `docs/`: each topic already lives in `docs/overview.md`, and the README links to it.
- The research table stays in the README as the project's evidence, with its caveats cut to two sentences.
- No claim was added: every recipe, path and review budget the README states was checked against `contracts/recipes/`, `docs/research/` and `internal/loomengine/template.yaml`.
