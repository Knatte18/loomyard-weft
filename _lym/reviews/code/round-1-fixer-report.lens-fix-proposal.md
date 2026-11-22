```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals (F1–F4) that the fixer carried out in `roleguard.go` and its test.

No findings that would make the fix wrong where it was right. Two cases are worth the fixer's attention:

- **LOW — first clone into a sandbox root that does not exist yet is refused.**
  - Changed text: the final `filepath.EvalSymlinks(filepath.Join(worktree, sandboxRootRel))` in `inOwnSandboxRoot`.
  - Case: a hardener runs `clone --into .lym/harden-sandbox/hub-1` before `.lym/harden-sandbox` exists.
  - What happens: `resolveThroughExistingAncestor` returns a path whose tail is missing. The walk still finds the candidate root. The final `EvalSymlinks` of the not-yet-existing root fails, so `err != nil` and the guard refuses.
  - Why it matters: the doc comment promises that a clone's destination "that does not exist yet counts where it would be made". That promise holds only once the root exists.
  - Evidence: the only non-test source of the sandbox root, `internal/hardenshed/sandbox.go:25`, builds the path and `internal/pairteardown/teardown.go:509` tests for it. I did not confirm that the hardener creates the root before its first clone.
  - The test rows always create the root first, so no row catches this.
  - A fix would compare against `resolveThroughExistingAncestor(Join(worktree, sandboxRootRel))` instead of strict `EvalSymlinks`.

- **LOW — a hub with no recorded shortname refuses its own sandbox.**
  - Changed text: the `ReadShortname(...)` check added for F4.
  - Case: a hub with no `.lym-shortname` record. `ReadShortname` returns `found == false` and the guard refuses.
  - Evidence: `hubgeom/parent.go:41` shows a hub can lack the record, and then no agent name can be formed (`ResolveParent` returns "no parent").
  - A strand name with a shortname normally implies the record exists, so this is probably unreachable. I did not check how strand names get their shortname when the record is missing.

The other proposals hold:
- **F1:** mapping the resolved `-weft` form back to its owner worktree, then requiring the owner's real sandbox root to equal the walked root, handles both the production link layout and a direct weft path.
- **F3:** resolving only the deepest existing ancestor and re-joining the missing tail is sound.
- **F2:** the rows look able to fail with their guarded behaviour broken.

Further calls:
1. `grep -rn "HardenSandboxDirName" internal` plus `sed -n 30,60p internal/hubgeom/parent.go`, one Bash call. My context did not show who creates the sandbox root, or what happens to a hub with no shortname record.
