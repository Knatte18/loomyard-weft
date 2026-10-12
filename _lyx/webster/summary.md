# Config layers and run settings: all batches done

Seven batches (covering plan cards 1–23) ran in order and each recorded `status: done`.

- **01 yamlengine-layer-ops:** `Merge`, `Strip`, `SetSparse`, `Prune`, `LeafPaths` and `AnnotateLeaves` in `internal/yamlengine/layers.go`; `configengine` layer geometry, `ValidateLocal` and the layered `Load` and `LoadOrTemplate`; `fabricengine.ConfigDirs`.
- **04 reed-layered-load:** reed loads the hub stack with its per-key classification.
- **05 shuttle-layered-load:** shuttle, orch, webster and logger load the hub stack.
- **07 batcher-layered-load:** batcher, its three loomshed rows, burler and models read their layers through `configengine`.
- **09 loom-layered-load:** loom, darn, landing, gate, board and fabric load their stacks; `repoWideFabricBase` is deleted.
- **13 configreg-local-bounds:** `LocalKeys` and `LocalDenied`, `PATTERN-config-layers`, `PrepareRunSettings`, the layered reconcile, the one-time sparse migration, and `lyx config --layer`.
- **18 config-resolved-provenance:** `lyx config --resolved`, `agentname.TaskSlug`, `run_settings` on board tasks, Seed-Child writing the pair's local overrides, the orch rule, and the layer stack in `docs/overview.md`.

## Reported deviations from declared file-ops

- **01:** `internal/yamlengine/set.go`, `internal/configengine/layers.go` and `layers_test.go`, `internal/configreg/fill_test.go`, `internal/fabricengine/junctionnames.go` and its test.
- **07:** shedrecipe, loomrecipe and loomshed fixture and seam-enforcement tests; `internal/loomengine/config_test.go`.
- **13:** `internal/hubreconcile/hubreconcile.go`, `internal/configengine/config.go` and `layers.go`, `internal/loomcli/wiring_test.go`, `internal/configcli/configcli_integration_test.go`.
- **18:** `internal/configengine/layers.go` and `layers_test.go`, `internal/boardcli/runsettings.go` and `verbs_test.go`, `internal/battenshed/deps.go`, three Seed-Child fixtures.
- **04, 05, 09:** none.

## Fork-reported rule breaches

Several forks edited files with `sed -i` or shell heredocs, against the edit-only rule; the forks said the results were reviewed and passed their gates.
Forks reported that the card gates list `./internal/*_test` paths that are not directories and dropped them.

## Verify gate fixes

Attempt 1 of the verify gate failed on two findings:

- `internal/loomcli.TestWire_EverySeamFilled`: four new Seed-Child run-settings seams were nil after `wire()`.
- `internal/pattern.TestOverview_RealFilePassesCheck`: the `PATTERN-hub-wide-config` entry line was 356 characters, over the 320 cap.

The fixer fork's commit `6f2beaba9` (`fix: list Seed-Child run-settings seams as batten-only and shorten the hub-wide-config PATTERN entry`) lists the four seams in `intentionallyNil` in `internal/loomcli/wiring_seams_test.go` and shortens the `PATTERN.md` entry to 282 characters.
The fixer reports `lyx gate test --tags integration` passing for `./internal/loomcli`, `./internal/pattern` and `./cmd/lyx`, and a clean worktree.
The fixer also ran `gofmt -w` on `wiring_seams_test.go`, which is a formatter run rather than an Edit call.

Attempt 2 failed on eight `internal/fabricengine` integration tests (correspondence index and warp anchor-walk).
The cause was that the card 15 clone walk puts a `Warp-SHA` trailer on the weft prime's reconcile commit and records a correspondence entry, so every `hubforge` fixture hub started with an entry no scenario asked for.
The fixer's commit `bdc29004f` adds `forgetCloneCorrespondence` in `internal/hubforge/hub.go`, called at the end of `cloneHub`: it rewords that commit to its subject and calls `RebuildIndex`. No test was changed.
The fixer reports `lyx gate test --tags integration ./...` with no failing packages and `lyx loom lint-comments` clean, did not rerun the default-tag tier, and used `git commit --amend` rather than an Edit call.
