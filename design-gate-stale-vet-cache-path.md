# gate tier1 fails on a stale Go vet cache entry naming a torn-down worktree

`gate-stale-vet-cache-path` · Task · bug, gate · draft

Imported from [issue #868](https://github.com/Knatte18/loomyard/issues/868).

Seen (2026-10-24): after gate-runs-once landed and its worktree was torn down, and the hub's tier commands moved from '-tags tier1,integration' etc. to single tags, every 'lyx gate test --tier 1' in another worktree (drop-transitional-tag-spellings) failed before any test with: internal/reedengine: can't parse raw cgo file: open /home/knatte/Code/loomyard-LYXHUB/gate-runs-once/internal/reedengine/proctree_linux.go: no such file or directory. Likely cause: under GOFLAGS=-trimpath a go vet cache entry for reedengine's cgo pass recorded the absolute path of the worktree it was built in, and a later vet under the same tag set matched that entry. Workaround: the orch ran 'go clean -cache' while no gate slot was held. Expected: a gate run never depends on another worktree's paths; lyx could key or isolate the Go cache per worktree, or detect the error and retry with a cleaned vet cache.