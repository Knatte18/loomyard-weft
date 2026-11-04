# A codexengine test writes the real ~/.lym/codex

`codex-test-writes-real-home` · Note · bug, codex, testkit

Found in the codex-trial-home-sandbox trial.

`~/.lym/codex/lym-trust.json` and `config.toml` held trusted roots under `/tmp/lym-1000/<id>/TestPrepare_AppendsStandingPassages<n>/001/main` and `.../wt`, rewritten at each gate run.

Cause: `TestPrepare_AppendsStandingPassages` in `internal/shuttleengine/codexengine/prepare_test.go` sets `engine.dirs.Board = t.TempDir()` after `newEngine(anchor)`. The fixture's codex.yaml, which points `home` and `operator_home` at the test's folders, is then no longer found, so the config resolves the template's `~/.lym/codex` and `~/.codex` and Prepare writes the real lym home.

Expected: the test keeps the fixture's board dir (or writes its own codex.yaml there); a test must never reach the operator's home.