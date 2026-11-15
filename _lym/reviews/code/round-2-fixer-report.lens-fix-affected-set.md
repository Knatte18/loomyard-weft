```yaml
lens: fix-affected-set
outcome: reported
```

No findings for the sets I checked.

The review findings whose premise is a set:

- **F2 (sites saying the arbiter cannot fix):** I searched the Go, markdown and YAML files for the stale phrases. Those were "edits nothing", "only moves are", "loom-engine recipe" and "arbiter … cannot". No arbiter-specific hit remains outside the three sites the fixer edited. The `orch-role.md` hit is about an orchestrator fork, not the arbiter. `shedarbiter/doc.go` and `loom-fragment-arbiter-fix.md` agree with the fix.
- **F1 (callers that build batten's arbiter deps):** `loadedArbiterDeps` has one caller, in `internal/battencli/drive.go:371`. `withTaskWorktree` sets `Worktree` and `Prompt.Worktree` before every `Judge` call. No other batten site wires `shedworktree.ArbiterSeam`.
- **F3 (stale lines in the template):** I did not rerun a search. My context shows only the three review-named lines in `loom-template-arbiter.md` changed, and the other lines in the section already defer to the edit rules.
- **F5, F6 (test coverage):** I did not rerun a search. Each is a single-test coverage gap, and the gate output shows `TestArbiterSeam_Changes` ran at tier 1.

Tool calls (2 of 6):
1. A grep across `.go`, `.md` and `.yaml` for stale "arbiter cannot edit" and "loom-engine only" phrasing. My context lacked a repo-wide view of F2's site set.
2. A grep for every use of `loadedArbiterDeps`, `Deps.Worktree` and `Prompt.Worktree`. My context lacked F1's caller set.
