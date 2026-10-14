# Card 16 — shuttle-seam-move

**Rename:**
- `//internal/shedadapters/singlellm.go` -> `//internal/shedadapters/shuttle.go`

**Move:**
- `internal/shedrecipe#writeStencilFile`

**Intent:** Two relocations ahead of card 17's deletions, with no behavior change.

`internal/shedadapters/singlellm.go` is renamed `internal/shedadapters/shuttle.go` with `git mv` and no other edit, since the file's lasting content is the `Shuttle` seam Bouncer and the Burler rows' attach probes keep (D10's "Out"); card 17 deletes the single producer's members from it and rewrites its header and the seam's doc comment.
`shedrecipe`'s test helper `writeStencilFile` moves from `internal/shedrecipe/entries_singlellm_test.go` to `internal/shedrecipe/fixture_test.go`, declaration and doc comment cut whole and pasted unchanged, since the `MultiLLM` tests and `minimalEntries` keep using it after card 17 deletes the SingleLLM test file.
