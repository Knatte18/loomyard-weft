# Reed sends a short launch-script line instead of the full command

All five batches finished with `status: done`, and the plan-level integration verify reported `OK`.

1. **shell-source-and-script-ext** (`034c4393f`): `internal/shell`'s `Shell` gained `Source` and `ScriptExt` for both dialects, with table tests.
2. **launch-script-staging** (`46ad25dbd`): `panebin.go` gained `stageLaunchScript`, which writes the composed line to `<AnchorPath>/.lyx/reed/launch/<guid><ext>` atomically and returns the source statement. A write failure degrades.
3. **launch-script-cleanup** (`40bec1b83`): `removeStrandLocked` deletes the scripts of every strand it removes, and `Down` removes the whole launch directory.
4. **send-launch-script-source** (`b7e4a576c`): `launchStrandLocked` types the source statement, and a failed add or unpersisted record deletes its script. `doc.go`, `CONSTRAINTS.md` and `docs/sandbox-howto.md` were updated. One informational deviation: `internal/reedengine/replace.go` was touched, because `ReplaceStrand`'s persist-failure path needed the delete. The fork could not confirm that the persist-failure tests fail without the new deletes.
5. **launch-script-live-test** (`73ecc9b94`): a live tmux integration test proves the sourced script runs the command and leaves the prelude's exports in the pane shell.

The integration fork ran `go build ./...`, `go vet ./...`, `go test ./...` and the integration-tagged `internal/shell` and `internal/reedengine` tests once, and all passed.
