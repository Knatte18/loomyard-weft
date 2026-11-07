# Settle the defects the comment cleanup found but left

The comment cleanup in `f7465bd8` changed only comments and prose.
It listed defects its workers found but did not fix.
This change settles each item on that list.

## What changed

- **`lymcwd.ResolveWithAnchor` is removed**, along with its private body `resolveWithAnchorCore`.
  Nothing outside the package's test comments called it, and every ungated caller holds a worktree root and uses `ResolveWorktree`.
  `gate_test.go` now pins `ResolveWorktree` as the one ungated resolver.
  `docs/overview.md` and `docs/shared-libs/lymcwd.md` no longer list the removed function.
- **The stale headings in `docs/shared-libs/configengine.md`** are fixed.
  - The `Load`, `LoadOrTemplate`, `Set` and `Edit` headings showed the old `(baseDir, module, template)` signatures; they now show the current ones.
  - The `_lym` token note is no longer filed as an exported function.
  - "Migration from old format" is renamed "Reconciling config files", which is what the section covers.
  - The layout drops task artifacts that no longer live in `_lym/` and adds the local layer's `.lym/config/`.
- **Files that more than one worker edited** were each read whole against their `f7465bd8` diff.
  That covers all of `internal/adoptcli`, every file in `cmd/lym` the cleanup touched, and `internal/websterengine/render.go`.
  - Boundary fragments became whole sentences: `pathrules_test.go`, `spawnobservability_test.go`, `stencilseed.go` and `verifiedtree_test.go` in `cmd/lym`, and `internal/adoptcli/deps_test.go`.
  - `cmd/lym/destructiveguard_test.go` named an allowlist exemption before the sentences it qualified; that sentence moved to the end of the block.
  - Claims the code contradicts are corrected:
    - three `cmd/lym` tests said they are integration-tagged, but they run at tier1 through `tiergate`;
    - `exitcode_test.go` said an unknown command prints cobra text, but it prints a JSON envelope;
    - `main_test.go` named `RunGit`, but the tests seed their repos with `gitkit.Git`;
    - `destructiveguard_test.go` said `repointLink` has no row in its table, but it has one;
    - `adoptcli/doc.go` gave the wrong order for the prime marks;
    - `render.go` said a blank `gateTiers` is refused, but it renders as `none`;
    - `render.go` named `plan/overview.md` where the recovery stencil reads `plan/00-overview.md`.

## Design decisions

- `ResolveWithAnchor` is removed rather than given a caller.
  No call site needs an anchor other than the recorded one, so keeping it would preserve an unused gate bypass.
- The stencil paths under `{{.specs_dir}}/loom/` are left unchanged because they resolve.
  `{{.specs_dir}}` renders to the hub's deployed specs directory, `fabricengine.SpecsDir`.
  `stencilstore` deploys each spec into a family folder named by its name's prefix, so `loom/loom-plan-spec.md` and its siblings exist there.
  Only the flat source directory `contracts/specs/` has no `loom/` folder.
  The cleanup worker most likely checked that source directory.
- `f7465bd8` names no file for its kept claims or boundary fragments.
  Those were looked for in the jointly edited files, and a scan of every Go comment block looked for repeated lines.
  Each repeated line it found closes a parallel list item on purpose, so none was changed.
- In the edited files, only whole `//` comment lines changed, apart from the `lymcwd` removal.
  No string, identifier or directive changed.
  Comment lines of 120 characters or more that no edit touched stay as they are.
