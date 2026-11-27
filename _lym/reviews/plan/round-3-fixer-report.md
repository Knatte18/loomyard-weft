---
dispositions:
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed-differently
    reason: "The line 136 and line 144 parts are fixed as proposed. The line 149 part is fixed differently: `anchored_hub_seeds_the_lines_under_the_anchor` is kept instead of deleted. The proposed `SeedWarpExclude(l)` call in `anchored_hub_wiring_seeds_the_lines_under_the_anchor` cannot fail. That subtest's `hubforge.AddPair` runs `Add`, which wires junctions (internal/fabricengine/add.go:673), so the `/backend` lines are already in the shared `info/exclude` before any later call. `seedWarpExcludeLines` skips a line that is already present (internal/fabricengine/warpexclude.go:36-37). So a `SeedWarpExclude` that wrote nothing would still pass the once-under-/backend and never-at-root checks, before or after `WireJunctions`."
---

## Changes

- F17, fixed: card 14 `_lym/plan/14-executor-loop.md`.
  - The tests are tier1.
  - Every `gitexec` or `gitrepo` error the tests use is now provoked in the real module against local git:
    - transport: `gitexec.Run` running `ls-remote` against a refusing origin URL;
    - rejected: `gitrepo`'s `PushRebaseFree` pushing a branch whose origin tip diverged;
    - auth refused: `PushRebaseFree` pushing to a bare repository whose pre-receive hook prints `Permission to hub denied to tester.`.
  - The engine's own error types and plain `errors.New` values stay built in process.
  - Cut "standing in for git" and "each error built in process".
  - The joined read-failure row wraps provoked transport errors.
  - `TestOutcomeClassOf` lists one row per class, taken from those sources.
- F18, fixed differently: card 15 `_lym/plan/15-snapshot-reader.md`.
  - Line 136 now says the covering row holds the refusal and the unmoved origin tips, but not the no-mutations check.
    That check guards no rule of its own: the refusal returns the literal `PushResult{}` before any push step runs (`pushanchored.go:210`).
  - Line 144 names `TestOwnership_RegisteredLinkedWorktreeKind`'s `RefusesMainWorktree` and `AcceptsPrimeWarpWorktree` as the holders.
  - Line 149: `anchored_hub_seeds_the_lines_under_the_anchor` is no longer deleted.
    Its bullet and the `TestSeedWarpExclude_SeedsWarpOnlyOnce` **Edit:** target are removed.
    The proposal to add a `SeedWarpExclude(l)` call to the wiring subtest was not made; the reason is in the frontmatter.
- Self-check, card 14:
  - **Uses:** gains `internal/gitexec#Run` and `internal/gitrepo#Repo.PushRebaseFree`.
  - The literal-want sentence makes one exception: a row wants a provoked error's text as that error's own message.

## Self-check

- fix-claims, MEDIUM, and fix-proposal, MEDIUM (merged duplicates): the added `SeedWarpExclude(l)` call cannot hold the deleted rule.
  Fixed with my own change. The forks proposed calling it before `WireJunctions`, which also fails, since `AddPair` already seeded the lines (`add.go:673`). The subtest is kept instead.
- fix-consistency, LOW, the literal-want sentence disagreed with the provoked transport text: fixed as proposed.
- fix-consistency, LOW, card 14's **Uses:** lacked `gitexec.Run` and `PushRebaseFree`: fixed as proposed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
