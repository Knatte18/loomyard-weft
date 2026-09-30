---
format: 5
approved: true
language: go
---

# Plan: reed sends a short launch-script line instead of the full command

Every strand pane reed realizes shows the full composed launch line — the pane-binary prelude chained onto the caller's opaque launch command — as typed input, which the operator finds noisy.
reed instead writes the composed line to a per-strand launch script at `<AnchorPath>/.lyx/reed/launch/<guid><ext>` and types only the dialect's source statement for it.
The pane shell runs the same statements in its own scope as before, and the full line stays inspectable on disk while its strand exists.
The change lands at `launchStrandLocked`, the one chokepoint every strand-realizing path goes through;
the launch command's content, shuttle, Selvage's pane, and `io.go`'s `Send` are untouched.

## Card Index

1 — shell-source-and-script-ext — add `Source` and `ScriptExt` to `internal/shell`'s `Shell` for both dialects
2 — launch-script-staging — `panebin.go` writes a strand's composed line to its launch script atomically and returns the source statement, degrading on write failure
3 — launch-script-cleanup — `removeStrandLocked` and `Down` delete the scripts of every strand leaving reed's state
4 — send-launch-script-source — `launchStrandLocked` types the source statement, a failed add deletes its script, and the docs describe the script route
5 — launch-script-live-test — a live tmux test proves the sourced script runs the command and leaves the prelude's exports in the pane shell

## Shared Decisions

- The deletion card (3) lands before the wiring card (4), so no commit in the sequence writes launch scripts that nothing deletes.
- `CONSTRAINTS.md`, `doc.go` and `docs/sandbox-howto.md` change in card 4, the commit that changes observable behavior, per the Documentation Lifecycle.
- The dialect everywhere is `shell.ForGOOS()`, the selector the prelude and the launch command already share.
- No existing test or tool matches pane text against the launch line's content; the tests that asserted the send-keys payload directly are retargeted in card 4.

## verify:

go build ./...
go vet ./...
go test ./...
go test -tags integration ./internal/shell/... ./internal/reedengine/...
