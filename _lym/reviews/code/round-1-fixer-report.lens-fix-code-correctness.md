```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code the fix changed. Verdict: three findings, none BLOCKING. I ran no tests.

1. **MEDIUM** — `internal/orchroute/walk.go#Destination`: a stopped or taken-down beam orch now holds its members' mail instead of passing it to the prime.
   - Input: beam `b` once ran an orch, then had `lym orch stop` or `BeamDown`, or the beam moved to another machine. A member of `b` posts a halt notice.
   - `stopOrch` (`internal/orchcli/stop.go`) only removes the reed strand. Nothing sets `State.Strand` back to `""` (the grep found no clear), so `Destination` still returns `b`.
   - The notice queues until that orch comes up on this machine, which for a beam moved elsewhere may be never. `ParentOrchRecorded` also reads true, so the driver counts as watched.
   - The review's F1 fix asked for "whatever its liveness", so this follows the proposal. The new text (`orchroute/doc.go`, `creel post --parent` Long, `summary.md`) says only that creel delivers the mail once the orch is live. It does not name the stopped or moved case. The fixer-report should.

2. **LOW** — `internal/loomcli/wiring_beam_test.go`, `internal/battencli/drive.go`: part of F7's premise is still unpinned.
   - The new assertion calls `c.tacklerPrompt()` directly, so it stays green if `goDriverDepsFor` stopped setting `Prompt: c.tacklerPrompt()`.
   - Nothing asserts `ParentName` on batten's arbiter prompt (`loadedArbiterDeps`, which F7 names as `drive.go:571`). Dropping `ParentName: c.promptParentName()` leaves every test green.

3. **LOW** — gate coverage of the changed routing: callers of the changed behaviour sit in packages the gate runs did not name.
   - `orchcli.NotifyFrom`, `NotifyFromScoped` and `NotifyAsker` are called from `internal/adoptcli`, `internal/burlercli`, `internal/reedcli` (which has `oomnotice_integration_test.go`) and `internal/webstercli`.
   - None of these four was in the tier 0 or tier 1 package lists, and neither was any repo-wide enforcement-test package. The new `loomcli` → `orchroute` import is therefore checked only as far as the named packages check it.

The other changed branches held on the inputs I traced: `Destination` with a load error, `admissibleOrchs`, `checkParentCaller`, `OrchChainAbove`, `promptParentName` and `runParentRecorded`. The new decision, drift, notices, chain and orch-routing rows would each fail with the old behaviour in place.

Tool calls:
- `git grep` for routing callers outside the gated packages and for any clear of the orch strand: my context had no repo-wide caller list for the `Notify*` functions.
