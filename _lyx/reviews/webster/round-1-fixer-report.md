# Webster round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit each.
After the last fix, `go build ./...`, `go vet ./...` (untagged, `smoke` and `integration`), `go test ./...` and `go test -tags integration ./...` all pass, and the working tree is clean.

## Fixed

- **F1 (MEDIUM)**, `769afcb6e`: `assertFilled` in `internal/configreg/fill_test.go` and `assertOneFillLine` in `internal/configengine/config_test.go` now assert `keys=<path>` instead of searching the whole log for the bare key-path, which the temp file path in the same line could satisfy.
- **F2 (LOW)**, `01892d549`: removed the `gitignore.Ensure is never called` line from `SpawnDriven`'s doc comment in `internal/ideengine/spawn.go`.
  The sentence before it already gives the reason (the child's commits never sweep up `.vscode/`).
- **F3 (LOW)**, `f12ad9136`: changed the new and changed comments in `internal/configengine/config.go`, `config_test.go`, `internal/configreg/fill_test.go`, `internal/loomengine/config_test.go` and `internal/websterengine/config_test.go` to one sentence per line.
  In `internal/loomcli/wiring_test.go` and `internal/webstercli/wiring_test.go`, only the changed sentences were re-broken.
- **F4 (LOW)**, `01d19b0c7`: `TestAddStrandUnless_LiveNamedSkips` now reads `reed.json` before and after the skipped add and asserts the bytes are identical, and it adds `kill-pane` to the forbidden tmux commands.
  As a mutation check, I temporarily added a `SaveState` call to the skip path, and the test failed; the mutation was reverted before the commit.
- **F5 (NIT)**, `e879da907`: a new `moduleTemplate` helper in `internal/configreg/fill_test.go` checks `configreg.Template`'s `ok` and fails with the module named, and all three fill tests use it.

## Deferred

None.
