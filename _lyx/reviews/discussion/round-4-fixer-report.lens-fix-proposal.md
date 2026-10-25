```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged F2's proposal as the fixer carried it out. F1, F3 and F4 are decision-class and were not fixed this round.

Proposals challenged: F2 (fixed partly as proposed and partly differently, on the `claimBoardEntry` clause).

1. **MEDIUM**: the changed D4 line "A first pass with no status file that `refuseStartOwnedElsewhere` refuses exits with the same code and stays a refused start under D5", together with acceptance bullet 345 as rewritten ("A full pass refused by any refusal that names another machine as owner … exits with its own code; dobby then pauses the run under the owner … sends the owner notice naming `lyx adopt`").
   - Summary: the reviewer's "any refusal" wording now makes a fresh start's refused first pass both a D5 refused start and a D4 owned-elsewhere pass, and two parts of the record give those cases opposite handling.
   - Evidence: D4 line 106 makes every pass that is not a continue pass a full pass, so a fresh start's first pass is a full pass. Bullet 345 then requires the owner pause, the lock release and the owner notice for it. The new D4 sentence and D5 line 202 instead require a refused-start notice and a backoff. D7 line 223, which the fix did not touch, says "A full pass refused as owned elsewhere is neither" a refused start nor a failing pass. So this case is a refused start in D4 and "neither" in D7.
   - Concrete case: dobby's local board shows no owner, and the pull inside `refuseStartOwnedElsewhere` (`owner.go:106`) finds another machine. One sentence says to pause under the owner and send the adopt notice; another says to post a refused-start notice and back off. The bullet and D7 line 223 both need to exclude a first pass with no status file, or the D4 sentence needs to drop "stays a refused start".

2. **LOW**: the changed D4 lines "through one typed error that `ownedElsewhereRefusal` … return[s]" and "A claim `claimBoardEntry` loses inside the Worktree-Create row stops that row like any row failure".
   - Summary: the proposed shared type is also what `claimBoardEntry` returns, so the two sentences point implementers in opposite directions for a lost claim.
   - Evidence: `claimBoardEntry` returns `ownedElsewhereRefusal(slug, owner)` (`owner.go:183`) from inside the `CreateWorktree` row (`wire.go:812`). If an implementer maps the pass's exit code with `errors.As` over the error the pass returns, which is the pattern `battencli` already uses at `arm.go:845` and `runsettings.go:42`, then a row failure that carries this error exits with the owned-elsewhere code, not as a row failure.
   - Concrete case: whether the row error reaches the pass's exit depends on the shed driver, which the record does not pin. The record should either say the row's error is not mapped to the code, or say that it is. A related point for the claims lens: line 157 says dobby never resumes a halted run, so "the run's next full pass" after a row stop that halts the run is a hand pass, not a pass dobby spawns.

Further tool calls:
- One grep of `errors.As`, `ExitCode` and `os.Exit` in `internal/battencli` and `internal/sheddriver`. I ran it to see how a pass's error maps to an exit code, which my context did not show.
