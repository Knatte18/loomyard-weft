# Card 33 — tmuxkit-socket-fixture

**Edit:**
- `internal/reedengine/contract_integration_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `internal/reedengine/mouse_boot_integration_test.go`

**Uses:**
- `internal/testkit/tmuxkit/tmuxkit.go`

**Intent:** Convert the tests that start tmux directly on a hand-rolled key with their own `kill-server` cleanup to `tmuxkit.Socket` (D17): every `lyx-contract-…-%d-%d` key in `contract_integration_test.go`, and the scratch servers torn down by a raw `kill-server` in the ensure-session and mouse-boot tests where the test, not production reed, chooses the key.
A test that exercises production reed's own key derivation (`reedengine.ServerName`, the standalone `lyx-<hash8>` key) keeps it, since that key is the subject under test, and relies on `tmuxkit.Main`'s directory for isolation.
Then find every test-side spawn that builds a child's environment from scratch (a `cmd.Env` not derived from `os.Environ()`) in packages that reach tmux, and pass `TMUX_TMPDIR` through, so no socket lands in the default directory (the decision record's open risk);
fixes go in the commit body.
This card changes no `smoke`-tagged file.

**ImpactSummary:** `internal/reedengine` integration tests only, plus any environment-from-scratch spawn found; production reed is untouched.
