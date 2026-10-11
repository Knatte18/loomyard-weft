# gate-test-hygiene: both batches done

Batch 01-gofmt-guard (head `d4a4c0609`) landed cards 1–5.
It added the gofmt guard to `cmd/lyx` and fixed the files it flagged, and added the headroom line, growth check and bytes-over-cap message to `internal/pattern`.
It also added `internal/patternlint` with the `lyx loom lint-pattern` verb, appended `lint-pattern` to the webster card gates, and renamed `darnMergeBase` to `pairMergeBase`.
Deviations were informational: test files and the `internal/loomcli` and `internal/websterengine` files whose rows or helpers had to follow the verb, rename and message changes.

Batch 06-round-gate-headroom (head `7bc69f2ef`) landed cards 6–10.
It ran the headroom lint in the Webster-Burler round gate with the whole-diff form as a told flag, and made the budget guard name the row value to set.
It added parent-pid, state and comm probes to `tmuxkit`, routed the test-side `/proc` readers through them, and trimmed `PATTERN.md` to 14725 bytes with detail moved into six new background files.
Deviations were informational: seam allowlist, tier-purity rows, and the `/proc` readers in smoke and integration tests outside the named glyphs.

## Verify gate fixes

The first verify attempt failed on `TestEnforcement_FabricVocabulary/tree-scan`: `internal/loomcli/wiring.go` held a bare weft token outside the owner set.
The fixer fork reworded that comment in commit `654b0e370` (`fix: drop bare weft token from loomcli wiring comment`).
Its reply says `lyx gate test --tags integration ./internal/lyxcwd ./internal/loomcli` passed and the worktree is clean.
