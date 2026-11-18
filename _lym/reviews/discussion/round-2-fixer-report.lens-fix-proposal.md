```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the reviewer proposals the fixer carried out as proposed (F15, F16, F17, F18, F19).

Findings:

1. **MEDIUM.** Changed text: D4, "A live run that started none reads as red", together with D2's new Preflight refusal, which checks `packages` only against the `harden-live` key's tiers.
   - Summary: this proposal can block convergence for good with no defect anywhere.
   - Case: a hub sets `harden-live-copies` to a tier other than `harden-live`'s. Preflight checks `packages` only against `harden-live`, so it passes. Each round the copies run starts no binary and the run "reads as red".
   - D4 says copies failures "never fail the gate". D6's CONVERGED needs "a green live gate", and the new D6 text bars auto-accept on "a round whose live report is red".
   - So the red copies run either blocks CONVERGED and auto-accept every round, or it is ignored. The fix doesn't say which, and Preflight doesn't check the copies key's tiers.

2. **MEDIUM.** Changed text: D2, "Preflight refuses a run whose `packages` hold no test of a tier the hub's `harden-live` key names".
   - Summary: the proposal is phrased over the list as a whole, so a mixed list keeps F16's hole open for each package that has no live-tier test.
   - Case: `packages` is `[internal/reedengine, internal/fabricengine]`, and only reedengine holds a tier-2 test.
   - Preflight passes, and the live gate runs green on reedengine alone. Nothing marks fabricengine as never verified: the live report lists binaries started, but only a run that started none reads red.
   - Evidence: the pattern says `{test_targets}` is filled "with the packages holding a test of the tier" (`pattern/PATTERN-test-speed.md:37`). A per-package refusal would close the hole.

3. **MEDIUM.** Changed text: D8's rationale, rewritten, and the deleted Open risks question.
   - Summary: the proposal leaves the record contradicting its own added arbiter entry, which the fix was told to leave untouched.
   - Evidence: the arbiter entry "D8's rationale quotes the 2026-10-16 treadle note as written" still orders D8's rationale to read "...whether this run may delete treadle is the open parent question in Open risks", and says to make that replacement "whatever the parent answers".
   - D8 now carries different text, and the Open risks question it points to is gone.
   - Added entries take precedence over the body for darn and drift (the prompt's decision-record rules). Both will still read the arbiter's text as binding.
   - Only a `lym loom decision replace` by the parent removes the conflict, and that is outside the fixer's write surface.

4. **LOW.** Changed text: D4, "A `--copies` call takes one n-slot lease, and that lease replaces its tier runs' own acquires … n concurrent runs of the key's rendered tier command".
   - Summary: the n-slot lease ignores a tier's own slot count, and the paragraph doesn't say what a key with several tiers does.
   - Evidence: a tier run acquires `run.tier.Slots` slots (`internal/gatecli/run.go:627`), not one.
   - If the live tier declares more than one slot per run, n copies in n slots run under-provisioned. That weakens the very bound the slot pool exists for.
   - The paragraph speaks of "the key's rendered tier command" in the singular, and doesn't say how a key naming two tiers is copied.

5. **LOW.** Changed text: D5, "`internal/pairteardown` also runs it … by spawning `lym harden sandbox clear` in that worktree".
   - Summary: nothing says what teardown does when `clear` refuses or fails.
   - Case: D5's verbs refuse outside a worktree "whose run's recipe is `hardener`". Take a pair whose run record is already gone or cancelled, but whose sandbox root exists: the refusal fires.
   - Neither D5 nor the new criterion says whether pairteardown then aborts the removal or goes on. If it goes on, the leftover path stays undeletable (`ignoredTaskPathDeletable`), which is the very case F19 raised.

Tool calls made:
- grep in `internal/gatecli` for `acquireSlot` callers and `watchdogApplies`: to check whether the slot count is per tier and whether the CPU watchdog defeats concurrent copies. My context had neither. The watchdog turned out not to apply, so there is no finding on it.
- grep of the hub's `gate.yaml` for tier `slots`: to see whether lym's tier 2 takes more than one slot. Nothing found.
- grep for a `Slots` field and the hub's `gate.yaml` location: a retry of the previous call, also nothing found.
