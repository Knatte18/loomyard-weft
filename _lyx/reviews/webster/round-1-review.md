---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/reedengine/overlay.go:236
    summary: EndSessionByName kills the tmux server when it ends the last session but never removes the socket file, so the engine-less teardown leaks the file card 19 and PATTERN-test-isolation say reed removes
  - id: F2
    severity: LOW
    class: consistency
    location: cmd/testtiming/redundancy.go:76
    summary: New Go comments are hard-wrapped mid-sentence instead of one sentence per line (PATTERN-comment-line-breaks, CLAUDE.md), in the new testtiming files, reedengine/doc.go, reedcli/watchdog.go, llmkit.go and the new gitoracle testmain file
---

Review range: `status.json` `product.parent` is `main`, and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `b32c30584`, 23 card commits plus the dot-fill fix).

Evidence gathered by running the tree, not only reading it:

- `go vet -tags integration|tmux|llm|integration,tmux,llm ./...` all pass.
- `go test -count=1 ./...` passes, including `cmd/lyx` (`tierpurity`, `tmuxisolation`, `llmtier`, `hermeticenv`) and `internal/pattern`.
- `go test -tags integration ./...` with a `tmux` shim that always fails passes, so no tmux-starting file is left in `integration` (card 14's detection rule, re-run on this tree).
- `go test -tags tmux -count=1 ./...` passes and leaves `/tmp/tmux-$UID/` and `/tmp/lyx*` unchanged.
- `go test -tags integration -race -count=2` over `fabricengine`, `verifyrun`, `verifytree` and `gitrepo/...` passes, so the newly parallel tests share no state `t.Parallel` breaks.
- `go run ./cmd/testtiming -redundancy -pkg ./internal/agentname -out <tmp>` (card 20's verify) exits 0 and writes a well-formed section.

Focus questions settled with no finding:

- Card 10: no retired entry ID (`PATTERN-test-tier-purity`, `-test-time-seam`, `-test-parallel`, `-hermetic-git-tests`, `-tmux-test-isolation`) remains outside `_lyx/`; the three deleted background files have no remaining link; each `pattern/` test file is named by exactly one entry.
  The remaining "Test Tier Purity Invariant" strings are prose invariant names, which the card says stay.
- Card 13: every per-package LLM-binary lookup is replaced by `llmkit.Claude` with the override its copy honoured (`LYX_REED_CLAUDE` for reedcli, burlerengine, shuttlecli and treadleengine, `LYX_SHUTTLE_CLAUDE` for orchcli, `""` for webstercli); no `smoke` build tag, local copy or `-tags smoke` survives in Go files or docs (the two `"smoke"` literals are plain strings).
- Cards 13-14: retagged files match the detection rule, and `internal/reedcli/smoke_test.go` carries `tmux || llm`.
- Cards 12 and 15: the guards cover `tmux` and `llm`, and the `llmkit` `os/exec` exemption is bounded to `LookPath` by `llmtier_test.go`.
- Cards 3-6: each shortened wait is a per-instance field or parameter (`reedCLI.watchdogTiming`, `watchdogTiming.ReapTimeout`, `ReapSession`'s `timeout`, `verifyrun.run`'s `delay`); production passes today's value.
- Card 18: no ad hoc `-L` key is left that starts a server without `tmuxkit` or a `down`-driven teardown that now removes the file.
- Cards 20-21: `TestClassify`, `TestParseProfile`, `TestRenderPackage`, `TestScanSpawns`, `TestRedundancyTags` and `TestResolveTags` cover the new mode without duplicating each other; spawn-driving tests of `cmd/lyx` and `reedcli` are listed under "no coverage" in the committed report.

### [MEDIUM] EndSessionByName leaves the socket file of the server it kills

**Location:** `internal/reedengine/overlay.go:236-244` (`endSessionByNameVia`), called from `internal/pairteardown/teardown.go:289`.

**Issue:** Card 19 and `PATTERN-test-isolation` ("tmux socket files") make reed remove the socket file of a server it tears down, and card 19 states that `Down` and "every other teardown reach it through that one method" (`ensureServerGoneLocked`).
`EndSessionByName` is engine-less, so it does not reach that method: when it ends the last session it runs `kill-server` and stops.
The comment above the call, "kill-server takes the socket with the server", is the premise card 19 disproved.
Every pair teardown whose worktree is already gone therefore still leaves a socket file in `/tmp/tmux-$UID/`, which is the leak the plan's socket cleanup exists to stop; focus item "every path that kills or tears down a tmux server removes its socket file" is not met.

**Fix:** After `kill-server` in `endSessionByNameVia`, remove the file for `socketKey` through `removeStaleSocket(socketDirFromEnv(), socketKey)`, waiting a bounded moment for the killed server to stop answering (the helper refuses a socket that still accepts a connection and `kill-server` is asynchronous), logging a failure at Debug without failing the teardown.
Correct the comment, state the removal in `EndSessionByName`'s doc, and cover it by adding an assertion to the existing `internal/pairteardown/teardown_integration_test.go` case that ends a pair by name (no new test).

### [LOW] New Go comments are hard-wrapped

**Location:**

- `cmd/testtiming/redundancy.go` (doc comments of `parseProfile`, `classify`, `coveringTests`, `measurePackage`, `runAlone`, `runPackageTimings`)
- `cmd/testtiming/spawnscan.go` (doc comments of `loadPkgSource`, `scanSpawns`, `analyze`, `collectLocals`, `kitSpawns`)
- `cmd/testtiming/main.go` (the doc lines added for `-tags` and `-redundancy`, and `resolveTags`)
- `internal/reedengine/doc.go` (the new "Socket-file removal" bullet)
- `internal/reedcli/watchdog.go` (the `ReapTimeout` sentence and the `dispatchReap` caller comment)
- `internal/testkit/llmkit/llmkit.go` (package doc)
- `internal/gitrepo/internal/gitoracle/testmain_test.go` (file header)
- `internal/gitkit/doc.go` (the changed "See PATTERN-test-isolation" sentence)

**Issue:** `PATTERN-comment-line-breaks` and `CLAUDE.md` require semantic line breaks, one sentence per line with a further break at an independent-clause boundary, and call fixed-column-wrapped comments legacy, not a convention to match.
The listed new or changed comments break mid-sentence at a column.

**Fix:** Re-break only those new or changed comments into one sentence per line; leave untouched legacy lines and the verbatim-moved comments in `internal/webstercli/cli_tmux_test.go` alone.
