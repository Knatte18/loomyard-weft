# Hardener live check passes packages go can resolve

`harden-live-bare-package-paths` · Task · bug, gate, loom · starting · Open-Window

## What was seen (harden-fabricengine, 2026-11-22)

The seed pass wrote `hardener.packages` as worktree-relative directories without `./`, such as `internal/fabricengine`.
`hardenLiveArgs` (`internal/loomcli/hardenwiring.go`) appends them verbatim to `lym gate test`, so go reads them as standard-library import paths:
`package internal/fabricengine is not in std`, then `[setup failed]` for each package.
The `harden-live` check after every fix round was red before any test ran, and the fixer looped on a gate no fix could clear.
The orch unblocked the run by rewriting the local layer with `./` prefixes.

## Fix

`hardenLiveArgs` passes each package as a directory path go reads as one: prefix `./` to a relative path that lacks it.
`CheckCampaign` already reads the packages as directories under the worktree root, so both forms stay valid in config.
A test pins that a bare `internal/x` reaches `gate test` as `./internal/x`.
