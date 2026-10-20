# Card 3 — editor-save-load-check

**Create:**
- `plan:internal/configcli#errHubFileLoad` -> `var errHubFileLoad = errors.New("hub config file would not load with the edit")`

**Edit:**
- `internal/configcli#copyStagedToHub`
- `internal/configcli#editHubWide`
- `internal/configcli/doc.go`
- `internal/configcli#TestDispatchHubWideBoard`
- `internal/configcli#TestDispatchLayers`

**Uses:**
- `internal/configcli#newHubFixture`
- `internal/configcli#seedModuleConfig`
- `internal/configcli#fakeEditor`
- `internal/configcli#fakeHubCommit`
- `internal/configcli#fakeSyncTracker`
- `internal/configcli#assertJSONErrContains`
- `internal/configcli#assertJSONOkContains`
- `internal/configcli#newLoadFailure`
- `plan:internal/darnengine#ErrTreeScopeWithoutTrees`

**Intent:** Load a hub-wide editor save through the module's own loader, as `setHubWide` does for `--set`, so D1's refusals hold on both write paths of `lyx config darn` and of every other hub-wide module.

`copyStagedToHub` runs `module.CheckLoad` over the board dir alone before it writes the staged bytes, and again after the strict `validateHubFile` passes.
When the after-load fails with an error other than the before-load's, compared by text as `newLoadFailure` compares, it restores the hub file through `rollBack` and returns an error wrapping both `errHubFileLoad` and the loader's error.
A file that already failed to load takes a save that leaves that same failure, as `--set` does.

`editHubWide` checks `errHubFileLoad` the way it checks `errHubFileChanged`: it keeps the staging copy and answers a refusal of its own, never `newLoadFailure`'s `--set` way forward.
The refusal names the loader's error, says the hub file is unchanged and names the staging copy's path.
When the loader's error wraps `darnengine.ErrTreeScopeWithoutTrees`, the input that reaches it being a save that empties `tree_scope` while `tree_scope_verify` stays non-empty, the way forward is to run `lyx config darn --set tree_scope_verify=` first, then remove the trees in `lyx config darn` again.
Otherwise the way forward is to copy the staging copy out of its path, re-open `lyx config <module>` and re-apply the edit from that copy with the named key fixed, because every open of the editor overwrites the staging copy with the hub file's bytes.
The guard this adds only refuses a save; it lets nothing new through, and a save onto a file already failing with the same error passes as before.

`doc.go`'s sentence that every `--set` write is checked through the module's own loader is restated to cover the hub-wide editor save too, with the kept staging copy and the editor refusal's way forward.

`TestDispatchHubWideBoard` gains a `module` column, empty meaning `board`, which names the module it seeds, dispatches, and reads the board file and staging copy of, and darn editor rows over a board `darn.yaml` that is the template with `tree_scope: {tools: ./tools/...}` and a non-empty `tree_scope_verify` holding `{targets}`, one row per saved edit:
- a save that empties `tree_scope` is refused with exit 1, one commit call, the board file unchanged, the staging copy holding the saved bytes, and an error naming the loader's no-tree text, the staging path and `lyx config darn --set tree_scope_verify=`;
- a save holding `tools` and `tools/` is refused the same way, the error naming both keys, the staging path and the copy-out way forward;
- a save that empties `tree_scope` and clears `tree_scope_verify` together writes the board file and removes the staging copy, as D1 allows;
- a save adding a `docs` tree writes the board file and removes the staging copy;
- over a board file seeded with a malformed key `./tools`, a save that changes only `writer` keeps that same failure and is written.

`TestDispatchLayers` gains darn rows on the default layer over a template board file: `--set tree_scope.tools=./tools/...` writes `tools: ./tools/...` under `tree_scope` and commits once; `--set tree_scope_verify=` with no tree is accepted; and `--set tree_scope.tools/=x` over a file holding `tools` is refused naming both keys and `newLoadFailure`'s `--set` way forward, the board file unchanged, which pins D1's way forward for a duplicate written through `--set`, the spelling the file already holds.
The generic rollback of a `--set` the loader refuses keeps its existing row, and the no-tree refusal's text is pinned by Card 2's `TestLoadConfig` row.

**ImpactSummary:** every hub-wide module's editor save gains a loader check that refuses only a save its loader newly rejects; the `--set` path and the per-worktree and local paths are unchanged.
