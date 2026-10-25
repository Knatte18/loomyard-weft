```yaml
lens: fix-consistency
outcome: reported
```

Findings, one directive:

**LOW** — `internal/loomcli/bootstrap.go:25` and `internal/loomcli/doc.go:22` still call the status strand "the session's overview window". The phrase is the one F2 removed from the orch row, so it could now be read as a twin of the overview window. These describe the loom checklist strand, not the batten overview, so the wording is not wrong. `internal/orchcli/runs.go:1` and `:36` use the same phrase for the `runs` strand, whose window is now named `beam: <slug>`. The phrase is inherited and was not touched by the fix, so I rate it LOW at most.

Checked and consistent:
- The orch row of `docs/overview.md` now matches the `internal/orchcli/runs.go:1` header (`beam: <slug>`).
- No other "its own first" claim about orch order remains in the repo. The remaining matches are unrelated.
- The new test row `ACyclicParentChainIsLeftOut` runs against the `parents` map and locations list that the fix extended with `loop-x` and `loop-y`. The prime's-window row was updated to include both loop locations, since the prime's window shows every orch.
- The `atOrBelow` comment says a chain that revisits a slug returns an error. `orchsAtOrBelow` already documents that a location whose chain does not read is left out, and it logs at Debug, so the two agree.

Tool calls (1):
1. `grep` across `*.go` and `*.md` for "its own first" and "overview window" phrasings, to find twins of the edited comments and doc text. My context held only the edited files, not the rest of the repo.
