```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked each sentence the fix added or rewrote against the code at HEAD.

1. **MEDIUM.** Card 9, the added clause "`New`'s doc in `internal/loomrecipe` justifies the status-path check by Shed's own status alone, dropping `tapestryPreflightEntry`".
   - The claimed justification does not hold. After card 9, no registry entry reads `Env.StatusPath` or `Env.StatusLockPath`, so the check compares a copy that nothing reads.
   - Shed itself reads only `paths.StatusPath`. So "Shed's own status" gives no reason for the env/paths comparison, and the rewritten doc would state a rationale the code does not have.
   - Evidence: the only non-test readers of `env.StatusPath`/`env.StatusLockPath` are `internal/shedrecipe/entries_simple.go:114-123` (`tapestryPreflightEntry`, which card 9 deletes) and the check itself at `internal/loomrecipe/loomrecipe.go:105-106`. `internal/battenrecipe/battenrecipe.go:32` already describes the case where no entry reads the copy, so there is nothing duplicated to guard.
   - Card 9's existing "`Env.StatusPath`/`StatusLockPath` stay (loomrecipe's coherence check reads them)" is circular in the same way.

2. **LOW.** Card 9, the added clause "`pattern/PATTERN-cwd-resolution.md`'s tiers end at `preflight.Check`".
   - The file also says "Three tiers: …" and "An orchestrator needs tier 3." (`pattern/PATTERN-cwd-resolution.md:21-23`).
   - With `CheckSeed` gone there are two tiers, and tier 3 no longer exists. The card only says the tiers end at `preflight.Check`, so it leaves the count and the orchestrator line stale.

Claims checked that hold:
- Card 6: `implementerEntry` has three way-forward texts naming `loom.implementer`.
- Card 6: the `engineMinimalConfig` doc comment is at `build_engines_test.go:22`.
- Card 6: the `TestDispatchLayers` local-layer row exists.
- Card 6: envkit's `TemplateSetting` resolves through `configreg.Lookup`, so registering `tapestry` answers `tapestry.implementer`.
- Card 6: `TestPerchHardCap` exists and already has a plan row.
- Card 8: the build row of darn's `darnVerifyGatedRows` has engine `DarnWrite`.
- Card 11: the reportengine darn setup sentence is at doc.go:132.
- Card 13: battencli never sets `DiscussionHistory`, so listing it keeps both directions of the seam test.
- Card 16: Record-Write's commit-when-uncommitted step makes the goto-then-resume route commit an existing pair.
- Card 17: the two commit closures have identical bodies, and no test drives either one.
- Card 5: loomcli/doc.go lines 26, 148 and 309 are the three sentences the card names.
- Card 9: preflight's existing `cancelErr` pattern means a cancel check after the seam is consistent with "a cancelled context wins".

Tool calls:
- Read `internal/preflightshed/preflight.go` lines 112-140: my context showed only lines 100-112, and I needed the cancel checks after the reconcile.
- Grepped cards 14 and 15 for `darnDeps`/`Commit`, and battencli for `DiscussionHistory`: I needed to know whether a later card moves the commit closure, and whether batten sets the seam.
- Grepped `internal/testkit/envkit/envkit.go` for the template lookup: I needed to know whether `tapestry.implementer` resolves once `tapestry` is registered.
- Grepped `internal` for readers of `env.StatusPath`: I needed to test card 9's "Shed's own status" justification.
