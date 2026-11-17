# Let a machine set fabric's branch_prefix over the hub file

Imported from issue #99.

`fabric.branch_prefix` could be set only in the hub file, committed in Bolt and shared by every machine of the hub.
`lym config fabric --layer machine` was refused because fabric had no machine key.
The local layer refused the key as cross-pair.
A branch prefix usually names the person who owns the branches, so one hub-wide value does not fit a hub that several people or machines share.

## What changes

- `branch_prefix` moves from the cross-pair category to the machine category in `fabricengine.ConfigSpec`.
  `lym config fabric --layer machine --set branch_prefix=<x>/`, run from the prime, now writes the untracked `<BoltDir>/.lym/config/fabric.machine.yaml`.
  The hub file stays the fallback, and the local layer still refuses the key.
  Every fabric load already tells both `Board` and `Anchor` through `fabricengine.ConfigDirs`, so every reader sees the machine value with no caller change.
- The machine category's definition widens from a shared resource bound or a fact of the machine to also cover a convention of the person working on the machine.
  The change is made in `configengine`'s `CategoryMachine` comment, its package doc and `PATTERN-config-layers`.
- The fabric template comment, the fabricengine package doc and `docs/overview.md` name `branch_prefix` as a machine key.

No config reinterpretation ships: a hub file's value keeps its meaning and is still read where no machine override exists.

## Design decisions

- **Per machine, not per user.**
  The machine layer lives under the hub's Bolt checkout on one machine, so it already is per user wherever one person works in that checkout.
  A separate per-user layer would add a sixth layer for no case the machine layer misses.
- **The other personal keys stay where they are.**
  The issue asked for a review of them:
  - The pull request's identity is gh's login, not lym config.
  - The Slack operator is never config: `lym slack setup` records it and the credential file is per machine already.
  - `batten.open_ide`, `seed_ide` and `open_window`, `landing.pr_reviewer` and `agents.efforts` are category None.
    They are open to the local layer and to a task's `run_settings` already.
    The machine category closes a key to both of those, so moving them would take away the per-worktree and per-run setting they have now.
  - A per-machine default for a category-None key would need the machine layer to admit category-None keys.
    That is a change to `configengine`'s layer rule, kept out of this change.
- **The beam this entry was planned behind has not landed.**
  The entry says it was planned after `adopt-build` lands; that beam (`adopt-build-resplit`) is still running.
  The change touches only fabric's key category and reads nothing the beam reworks, so it does not wait.

## Open risks

- A pair made on one machine under prefix `a/` and later looked up on a machine with prefix `b/` after all its local parts are gone:
  `Topology.PairTaskBranchOrOrigin` tries the current prefix's name and the bare slug, so it misses `a/<slug>` on origin.
  The same gap exists today when the hub-wide prefix changes.
  Teardown carries the branch names it read when it started, so a teardown already running is unaffected.

## Tests

- `TestLoadConfig` gains a case: a machine override's `branch_prefix` wins over the hub file's while the hub file's `pathspec` is kept.
- `TestModules_EveryTemplateKeyClassified` pins fabric's machine keys as `branch_prefix`.
- The configcli row that pinned the refusal of `--layer machine` on a hub-wide module with no machine key now uses `slack`, since fabric has one.
