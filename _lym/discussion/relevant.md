# Relevant files

- `docs/research/speed/tier1-test-revision-2026-11-06/report.md` — the measured levers, sizes and fixtures this split takes
- `internal/hubforge/template.go` `Shape`, `Shapes`, `templateFor`, `requireIntact`, `CopyHub`, `commitRelocatedBinding`, `SharedHub`, `hubAt`, `relocateHub` — the template build, copy and fingerprint D2 and D3 change
- `internal/hubforge/hub.go` `markFixtureBuild` — the ref-counted fixture-build mark concurrent builds share
- `internal/fabricengine/post-checkout.sh` — the hook script whose `dirname` and `basename` D5 replaces
- `internal/fabricengine/hook.go` `InstallPostCheckoutHook` — holds the chain wrapper that runs `dirname` on `$0`
- `internal/fabricengine/mergein_integration_test.go` `newMergePairFixture`, `commitOnBranch` — already a plain copy; its layers are D7's candidates
- `internal/fabricengine/livestate_matrix_test.go` `runCell` — the cross product's cell, where D6 folds and D7 templates the arrange
- `pattern/PATTERN-leaf-packages.md` — `internal/dotgit`'s closed imports and sole-parser role
- `pattern/PATTERN-gitrepo-client-boundary.md` — local reads belong to go-git and `dotgit`
- `internal/toolfloor/toolfloor.go` `GitFloor` — the oldest git lym runs on
