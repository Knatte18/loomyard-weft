# Implementing batch 01-run-watch-setting

Card 1 creates `LookupRunWatch`, whose first production caller is card 4, so the test-only-code guard failed card 1's gate.
I added a temporary `testOnlyCodeAllowed` entry for `internal/dobbyengine.LookupRunWatch` in `cmd/lym/testonlycode_linux_test.go` (with a `Loosened-Check` trailer).
The guard fails on a stale entry, so the card that wires the lookup must remove it; card 4's targets do not list that file.
