```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set — per change, the set I found and whether the diff covers it.**

1. **`shedrecipe.Env.CreateWorktree` and `NewWorktreeCreate` take a second `HoldPrimeLock` argument.**
   - Production set: `wire.go:941`, `entries_batten.go:55`, `recipe.go:342` and `envkit.go:500`.
   - Test set: the `battencli` tests, `shedrecipe/fixture_test.go`, and the `battenshed` create and stuck tests.
   - All are covered. The grep over `internal` and `cmd` turned up no other caller, `loomcli` included.

2. **`refuseHeldMember` gains a `holdPrime` parameter.** Both callers, `wire.go:958` and `wire.go:1000`, are updated, and `TestRefuseHeldMember` is updated. Covered.

3. **`fabricengine.AddOptions.Hold` is new.**
   - `Add` is the only consumer, and its doc comment is updated. The new `Hold` field is general and names nothing of batten, so `PATTERN-fabric-isolation` holds.
   - `PATTERN-fabric-git` is untouched: `Add` still pushes nothing.
   - Covered.

4. **The warp and weft branch-exists reads and both probes move above the hold.**
   - I found no other reader or asserter of the old order. `docs/overview.md` and the `Add` doc comment now say the probes start first.
   - The `add.go:303` comment, "Probe both origins before the first mutation", is kept in the new position. Covered.

5. **The prime lock's span in Worktree-Create narrows from the whole closure to the `Add` pre-flight and the held-member removal.**
   - Docs:
     - `docs/overview.md:749` still says Create and Teardown "wait for a prime lock another run holds". That stays true, and the new line at 750 states the narrowing.
     - `battenshed/doc.go` and the `recipe.go`, `deps.go`, `create.go` and `wire.go` comments are updated.
   - Not updated: the `primelockwait.go:1` file header and the `acquirePrimeLock` comments. They say "both prime-lock producers … call" it, which is no longer true of Create's direct call. Create now reaches it through its hold. The text is still loosely accurate, so I rate it LOW.
   - PATTERN: no entry names the prime lock's span for Create. `PATTERN-fabric-git` and `PATTERN-owner-machine` hold no such claim.
   - Fabric doc: `fabricengine/doc.go:109` and `originreconcilemoves.go` describe the origin reconcile's try-acquire of the prime lock. A create holding the lock for less time only makes the reconcile less often busy, so they stay correct.

6. **The `HoldPrimeLock` error contract, `PrimeLockBusyError` and `primeLockFailure`.**
   - The only consumers are the `Call` mapping in `create.go` and `createRefusal`. `createRefusal` passes unknown errors through verbatim, so a busy or failure error from `Add`'s `Hold` survives it unchanged, because `Add` returns it as is.
   - The closure's other paths (`refuseHeldMember`, and `claimBoardEntry` and `taskWorktreeComplete` before any hold) return the error untouched.
   - Covered.

7. **Declared PATTERN changes.** The directive's constraints section lists no declared changes, and the diff touches neither `PATTERN.md` nor `pattern/`. Nothing is owed.

8. **Commit messages.** There is no `Pattern-Departure:` trailer, and none is needed.
   - The `fix:` commit carries only the attribution.
   - The first commit says "the row now hands its create closure a HoldPrimeLock". That matches the diff.

9. **`_lym/landing/summary.md` against the diff.** Every claim I checked holds. One claim is not in the diff: that a board note `push-holds-remote-refs` was filed. It lives outside the diff, so I treat it as unverifiable and raise no finding.

10. **Stencils and specs.** I found none that describe Create's lock span, so none are affected.

**Findings**

- **LOW, consistency** — `internal/battenshed/primelockwait.go:1` and the `acquirePrimeLock` doc comments (lines 21–26)
  - The file header names "both prime-lock producers (Worktree-Create and Worktree-Teardown)" as callers of `acquirePrimeLock`. Create now calls it only inside the hold closure, once per hold, possibly several times per run.
  - Evidence: `create.go` `hold` closure, and `primelockwait.go:1`.
  - The fix is a wording touch-up: the file header should say that Create reaches it through its hold. Dropping it is also defensible, since it is stale only in nuance.

I found no other member of the affected set left wrong.

**Tool calls:** 1 call, a grep for "prime lock" across docs, `fabricengine/doc.go`, `originreconcilemoves.go`, `battencli`, `entries_batten.go`, `PATTERN.md` and `pattern/`. My context lacked docs or comments outside the diff that describe the lock's span.
