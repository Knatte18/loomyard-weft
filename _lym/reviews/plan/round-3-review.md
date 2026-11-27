---
verdict: BLOCKING
findings:
  - id: F17
    severity: BLOCKING
    class: consistency
    location: _lym/plan/14-executor-loop.md:71
    summary: "Card 14's tier0 tests hand-write other modules' errors, which PATTERN-test-stand-ins bars: TestRunRule's transport row and runners return made-up git failures standing in for git (lines 64 and 71), and TestOutcomeClassOf builds a *gitexec.GitError with a chosen stderr and wraps gitrepo.ErrPushRejected (line 76)."
    fix: "In card 14, make `TestRunRule` and `TestOutcomeClassOf` tier1 (line 63) and have every `gitexec` or `gitrepo` error they use come from the real module. Transport: `gitexec.Run` of an `ls-remote` against an origin URL that refuses the connection. Rejected: a `gitrepo` push of a branch whose origin tip diverged. Auth refused: a `gitrepo` push against a bare repository whose pre-receive hook prints `Permission to hub denied to tester.`. Errors of the engine's own types (`*AnchorStopError`, `*destructiveRefusal`, `*fieldReadError` wrapping a provoked error) and plain `errors.New` values may stay hand-built. Cut the words `standing in for git` and the `each error built in process` clause. No target changes."
  - id: F18
    carries: F15
    severity: MEDIUM
    class: design
    location: _lym/plan/15-snapshot-reader.md:149
    summary: "The shrink is now shown, but not every named deletion has the holding test or reason F15 asked for: anchored_hub_seeds_the_lines_under_the_anchor is the one internal/fabricengine check that SeedWarpExclude seeds under a subpath anchor, and the sibling card 15 names never calls SeedWarpExclude; two other reasons (lines 136 and 144) name holders that do not hold what they are said to."
    fix: "In card 15 line 149, keep the deletion and add that `anchored_hub_wiring_seeds_the_lines_under_the_anchor` gains one call, `fabricengine.SeedWarpExclude(l)`, after `WireJunctions`; its existing once-under-/backend and never-at-root checks then hold the deleted rule. In line 136, say the covering row holds the refusal and the unmoved origin tips but not the deleted subtest's no-mutations check, and say why that check guards no rule of its own. In line 144, name `TestOwnership_RegisteredLinkedWorktreeKind`'s `RefusesMainWorktree` as the holder of the registered-linked refusal, beside `AcceptsPrimeWarpWorktree` for the warp-checkout call. No target changes."
prior:
  - id: F15
    status: open
    answer: "The estimate, the further deletions and the removal of the halt clause are in. What stays open is the per-deletion justification the fix required, carried as F18."
  - id: F16
    status: resolved
---

### [BLOCKING] Executor tests fake gitexec and gitrepo errors to stay at tier0

**Location:** `_lym/plan/14-executor-loop.md:71`, with lines 63, 64 and 76.

**Issue:** `PATTERN-test-stand-ins` says a test never uses made-up data that simulates another module, and that an error of another module is provoked in the real module.
`PATTERN-test-speed` says another module is never faked to lower a tier.
Card 14 plans tier0 tests whose "runners return each row's error, standing in for git",
whose read-failure row needs hand-made transport errors,
and whose class table builds a `*gitexec.GitError` with a stderr written to satisfy `IsTransportFailure`.
`outcomeClassOf` interprets that text, so no exemption in the entry applies,
and `docs/research/no-fake-other-modules.md` admits no hand-built error.
All three failures can be provoked with local git:
`internal/fabricengine`'s tests already hold a refused origin URL and a permission-refusing pre-receive hook.

### [MEDIUM] Some card 15 deletion reasons do not hold (carries F15)

**Location:** `_lym/plan/15-snapshot-reader.md:149`, with lines 136 and 144.

**Issue:**
- `anchored_hub_seeds_the_lines_under_the_anchor` calls `SeedWarpExclude` on a hub anchored at `backend`.
  Its named sibling calls `WireJunctions`, which reaches `seedWarpExcludeLines` by its own call site.
  `SeedWarpExclude` passes `l.AnchorRel` itself, and `EventFor` depends on it.
  The other subtest that calls it runs at anchor `.`.
- `TeardownLockHeldPastTheWaitOwesTheRecordsPush` discards the push result,
  so it does not hold the deleted subtest's `res.Mutated().Len() == 0` check.
- No earlier subtest of `TestOwnership_WarpCheckoutKind` calls the registered-linked check;
  that refusal is held by `TestOwnership_RegisteredLinkedWorktreeKind`.

## Focus answers

- Net shrink: the plan shows it before any card runs, and no step finds it out or halts on it.
  My line count of the named deletions is in line with the plan's figures for cards 2, 3, 4 and 15, and above its added-line budget.
  Card 15's commit body reports the totals and does not halt.
  The deletion reasons that do not hold are F18.
- Read-failure contract: store order, the Store per field, `Snapshot.Phases` and the re-read's Phase are now stated.
- Ref-storage test: not a defect.
  The `FieldLocalRefs` rows now read from the main worktree and the linked worktree's root,
  which `gitrepo`'s open accepts.

## Rejected

Observations below this incremental round's bar for a new finding, which is BLOCKING:

- `FieldAnchors` (card 15) reads `committedMarker` in every records worktree `List` returns, a prunable one included,
  where the read fails on the missing directory.
  The card does not say to skip prunable entries.
- Card 15 does not say whether a failed field stops the later fields of the same phase.
  If the reader continues and nests joins, card 14's one-level unwrap drops failures.
- No planned test asserts the reader's walk past a failing store,
  or the Phase recorded for a transport failure on a re-read.
- `actionRunner` returns only an error, so a seam call's successful answer (D3) has no way into the snapshot,
  and the plan does not say which part adds one.
