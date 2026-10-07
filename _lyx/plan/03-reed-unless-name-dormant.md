# Card 3 — `--unless-name` matches a dormant strand

**Edit:**
- `internal/reedengine/strand.go`
- `internal/reedengine/strand_test.go`
- `internal/reedcli/add.go`
- `internal/reedcli/unlessname_integration_test.go`
- `internal/vscode/config.go`

**Intent:**
`AddStrandUnless` skips the add when a strand named `unlessName` exists in this worktree's reed state, live or dormant (D4, #408).
Today it skips only on a live, visible match (`liveStrandNamed` over the alive pane set), so a folder-open on a prime whose orch strand is dormant stacks a bare claude where the orch belongs.

`liveStrandNamed` is replaced by a plain name match over `st.Strands`, with no liveness or anchor condition, so the pane listing that fed it is no longer needed on this path.
On a match the call still returns that strand and true, having saved no state, reconciled nothing, launched nothing and moved no focus; the dormant orch comes back only through `lyx orch start`, as today.
Bound: the skip only ever suppresses an add; it launches, revives and removes nothing.

The `--unless-name` flag's help and the `add` verb's Long text in `add.go` say the match is any strand of that name in this worktree's state, live or dormant.
The comment on the vscode chain's add row in `config.go` says the same.

`strand_test.go`'s table for the old helper becomes the name-match table, and `TestAddStrandUnless_NotLiveAdds` flips: a dormant orch strand in state now skips, and only a state with no orch strand adds.
The tmux-tier `unlessname_integration_test.go` gains the dormant case (an orch strand whose pane was killed: `reed add --unless-name orch` adds nothing) and keeps the no-orch case adding.

**ImpactSummary:** One caller (the vscode folder-open chain) now also skips over a dormant orch; every other add is unchanged.
