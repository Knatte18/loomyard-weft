# operator-surface-fixes: all eight cards landed

Every batch reported `done` with no deviations from the declared file-ops; `record-batch` returned only informational scope warnings.

- 01 `yaml-fill-missing`: `yamlengine.FillMissing`, the in-memory merge of a file's missing template keys, with tests and a `docs/shared-libs/yamlengine.md` update (head `4afb8637c`).
- 02 `config-load-fills`: `configengine`'s shared load fills missing keys and logs them instead of refusing; test updates across the config-bearing packages, a new `internal/configreg/fill_test.go`, `docs/shared-libs/configengine.md` and `CONSTRAINTS.md` (head `ecdedd46b`).
- 03 `reconcile-hints`: reed config templates, `reed up` help text and `docs/overview.md` reworded to the default-until-reconcile wording (head `cf00e0199`).
- 04 `reed-unless-engine`: `AddStrandUnless` in `internal/reedengine`, skipped while a named strand is live (head `dc6680473`).
- 05 `reed-add-unless-flag`: `lyx reed add --unless-name` with a `skipped` envelope, plus a real-tmux integration test (head `34c219027`).
- 06 `ide-spawn-clean`: `--unless-name orch` in the chain, `tasks.json` regenerated on every call, `.vscode/` excluded via `info/exclude`, `internal/vscode` no longer touches `.gitignore`; docs and `CONSTRAINTS.md` updated (head `0bc5b150e`).
- 07 `gitignore-pkg-delete`: `internal/gitignore` deleted with its doc entry (head `8db756d50`).
- 08 `repo-housekeeping`: the repo `.gitignore`'s mill block became a plain section, and the CLAUDE.md TEMPORARY items for #334, #339 and #340 were dropped (head `8780eafce`).

The integration fork ran the plan's `## verify:` once at `8780eafce` and reported `status: OK`.
