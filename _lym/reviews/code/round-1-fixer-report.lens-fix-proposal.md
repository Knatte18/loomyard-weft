```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3 as the fixer carried them out.

**Findings**

1. **LOW** — F2's claim that the `pairStart` nil default and the `StartDobby` wiring "need no test of their own". The changed text is `bringUpLocation` plus the `battencli/cli.go:589` call `orchcli.BeamUpWorktree(c.location, parent)`.
   - Summary: the new test pins only `bringUpLocation`'s own rule, not what the callers pass it.
   - Evidence:
     - `bringUpLocation` returns `prime` only when `worktree == prime.WorktreeName`.
     - The batten call passes `c.location`, the receiver's own location, which is probably the task pair for `c.slug`. The comment at `cli.go:155` says "over this receiver's prime".
     - If `c.location` is not the prime and `parent` is the prime's name, the name check fails.
     - The code then resolves `fabricengine.WorktreePath(c.location, "<prime>")`. This is correct only if `WorktreePath` derives the sibling path from the hub.
     - `pawlwiring.go` passes a real prime from `prime()`.
   - Impact: no test covers the batten wiring, so a wrong receiver location would go unnoticed. I did not confirm that `c.location` is non-prime or what `WorktreePath` does with it. Treat this as a doubt about the proposal, not a demonstrated defect.

**Proposals checked with no breakage found**

- **F1** (`startDobbiesThroughBeamStart` with an injected run):
  - The label rule (`root` for the prime) moved intact.
  - The test fails if the `root` mapping, the continue-on-failure behaviour or the output in the joined error is dropped.
  - `beamStartChild` keeps the `testing.Testing()` guard, so the exec path stays untested. That is acceptable, since it is thin.
  - `cmd.Output()` captures stdout only. This matches the review's note that the JSON error envelope goes to stdout.
- **F3** (doc rewording): the sentence now describes the dobby start as the first step of the pair. No sibling in `doc.go` contradicts it. Line 13 still says `BeamUpWorktree` "returns a dobby that was refused or failed as an error", which agrees.

**Further tool calls (1)**
- `sed -n` of `internal/battencli/cli.go:575-600` and `internal/loomcli/pawlwiring.go:105-125`. I read these to see which location each caller passes to `BeamUpWorktree`; my context held neither call site.
