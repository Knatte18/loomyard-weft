# config: let a machine set fabric's branch_prefix

`fabric.branch_prefix` can now be set per machine with `lym config fabric --layer machine --set branch_prefix=x/`.
The machine value overrides the hub file, which stays the fallback.
Before, the key was cross-pair: only the hub's default layer, committed in the board repository, could set it.
That layer applies to every machine and user of the hub, but a prefix usually names the person who owns the branches.

Closes #99

## Design decisions

- `branch_prefix` moves from `CategoryCrossPair` to `CategoryMachine` in `fabricengine.ConfigSpec`.
  The local layer still refuses it.
  The `configengine` docs, `PATTERN-config-layers.md` and `docs/overview.md` now count a person's convention, such as a branch prefix, among machine keys.
- A pair made under another machine's prefix must still be found here.
  `PairTaskBranchOrOrigin` and the origin lookups behind it now fall back to the one origin branch ending in `/<slug>` whose records branch the weft origin also holds.
  Requiring the records branch keeps an unrelated branch ending in the slug from being taken.
  Two such branches are an error naming both.
- `Add` with `AdoptOnly` takes the warp branch from the pair instead of building it from the current prefix, so an adopted pair keeps its name.
- `slugOfBranch` trims everything through the last `/` from a branch that lacks the current prefix, since a slug holds no `/`.
- In the remote warp cleanup, an archive tag counts as evidence of a fabric-managed branch only for a name this machine's prefix produces.
  For any other branch the slug is a guess at where the prefix stops, so a tag of that slug proves nothing.
- The other personal keys keep their layers.
  The pull request identity is gh's login, and the Slack operator is recorded by `lym slack setup` in a per-machine credential file.
  `batten.open_ide`, `seed_ide`, `open_window`, `landing.pr_reviewer` and `agents.efforts` stay category None.
  The machine category closes a key to the local layer and to a task's `run_settings`, so moving them would remove the per-worktree and per-run setting they have now.
  A per-machine default for category-None keys would need a change to the layer rule in `configengine`, which this task leaves alone.

## To check by hand

No regression check or integration suite ran for this task.

- The review of the other personal keys is recorded only as the decision above, in no code.
  Check that leaving them in category None is the call you want.
- `pairbranch.go`, `originOtherPrefixBranch`: the other-prefix lookup reads every remote branch tip of the warp and weft origins.
  Check that the cost on a large origin is acceptable.
- `slugOfBranch` in `pairbranch.go`: a branch with a `/` in a bare pre-prefix name, such as `feature/x`, now names slug `x`.
  Check that no pair is named that way.
