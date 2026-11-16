```yaml
lens: guard-reach
outcome: reported
```

**guard-reach.** I checked whether each guard, refusal, branch and test row the cards add can be reached from a real entry point.

**Per guard: reachable or not**

Card 1
- `HostnameName`'s error for an unreadable or empty hostname: reachable through `nameFromHostname`. The added `TestFromHostname` assertion reaches it only "error for error" on the host it runs on, which is enough.

Card 2
- Replace selects the prime when `store.primeOwner` is in the set: reachable, through the row `ReplaceSetRewritesOpenBeamsAndPrimeUnderAFormerName`.
- Replace selects an open beam under a recorded owner: reachable, through the same row and `ReplaceSetSelectsAnOwnerOnlyOriginHeld`.
- A done beam is skipped: reachable, through the new `landed` seed.
- An empty name matches nothing: reachable, through `EmptyReplaceNameMatchesNothing`. With prime `""` and `beamer` unowned, the guard is load-bearing.
- A beam with no recorded owner is never selected: reachable, since `beamer` stays empty.
- Replace overlapping `req.Prime` or `req.Slugs` ("written once even when req.Prime also asked for it"): **not reachable from any caller**. Card 4 sets only `Machine` and `Replace`, and no row combines them.
- A replace-selected entry reported in `Lost` after a competing origin write: reachable in principle, through a push rejected and replayed with origin winning. **No named test reaches it** for a replace-selected entry.

Card 3
- `CheckName` refusal: reachable, through a row of `TestNameRefusesWithoutWriting`.
- Refusal when `FilePath` returns ok false or an error: reachable through the seam. The named row is "a config directory that does not resolve", and the relative-variable error case is not named.
- `showName` bad-file branch: reachable (tier0 row). Its no-config-directory branch is also reachable (tier0 row). `showName` with a `FilePath` error is **unspecified and untested**.
- The cwd hub in `sweptHubs`, and its dedup by path: **no named row reaches it**, since every row hands in hubs through `Hubs()`. The case of `lymcwd.Resolve` failing inside a hub is unclassified.
- `pullOwners` with a pull error or a non-empty skip, marked "checked locally only": reachable. The unreachable-origin row goes through `BoltSkipFetchFailed` (`bolt.go:320`).
- `pullOwners` when `ReadOwners` fails: **no branch is specified**.
- `checkCollision` refusal: reachable (the `neve` row). The success of a hand-broken name whose `names` ends in `osl46` is reachable. The conditional way forward for "repair back to a lost name" **has no state `checkCollision` can see**.
- `rewriteFile` decoded-with-invalid-name, undecodable and empty-entry branches: reachable (rows).
- A read error from `ReadFile`, distinct from a decode error: **no branch is specified**.
- The no-write branch when the name is set and `names` is unchanged: reached by card 4's re-run row, but nothing asserts it.

Card 4
- `failed` hub: reachable, through the unreachable-origin row.
- Re-run sweeps the failed hub: reachable.
- `nothing to sweep`: reachable (row).
- Recoveries appear once any hub was swept or pushed: reachable (row).
- Non-zero exit on a failed hub: reachable. Non-zero exit on a lost entry: **no test reaches it**.
- Per-entry `Lost` report: **no test reaches it**. A competing write needs the `BeforePush` seam, which `machinecli` does not expose.
- `pushed`: reachable in production. A run whose earlier sweep committed but did not push leaves the Bolt ahead; the re-run writes nothing, and `pushClassified` pushes the ahead branch (`pushclassified.go:120-126`). **No `TestNameSweep` row reaches it**: the unreachable-origin row fails at the fetch (`bolt.go:320`), so no local commit is left.

**Findings**

1. MEDIUM, design. Card 4 (`04-owner-sweep.md`, sweep step 5 and the `TestNameSweep` rows): no row reaches the `pushed` report.
   - It needs a hub whose earlier sweep committed and then failed at push, which the row list cannot produce: its unreachable-origin row fails at the pull.
   - Evidence: `internal/fabricengine/bolt.go:320` (fetch failure is a skip, so no commit); `internal/boardengine/ownerset_integration_test.go:331-338` (`TransportFailureAtPush` shows the needed setup: the origin breaks after the commit); `internal/fabricengine/pushclassified.go:120-126`.
2. MEDIUM, design. Cards 2 and 4 (`02-owner-replace-set.md` lines 31 and 34-38; `04-owner-sweep.md` step 4 and the exit rule): no named test reaches the `Lost` path for a replace-selected entry.
   - Card 2 adds no row with a competing origin write under `Replace`. Card 4's per-entry lost report, and the non-zero exit it drives, are reached by no row.
   - Evidence: `ownerset.go:141-160` (`Lost` comes from the post-push re-read over `written`); `ownerset_integration_test.go:453-470` (`CompetingWriteLosesToOrigins` uses `Slugs` only).
3. MEDIUM, design. Card 3 (`03-machine-name-verb.md` step 4 and the test lists): the cwd branch of `sweptHubs` is reached by no row.
   - Neither a hub only the working directory names nor the dedup with a registered hub is covered.
   - `lymcwd.Resolve` fails from a hub subdirectory (`ErrCwdOutsideAnchor`) and from the hub container (`ErrNotAGitRepo`). The card files both under "a directory in no hub adds none". So a run from inside an unregistered hub, other than at a worktree anchor, silently sweeps nothing. D6's bound names exactly that run as the way forward.
   - Evidence: `internal/lymcwd/lymcwd.go:73-86` (`Resolve` requires cwd to equal the anchored directory).
4. LOW, consistency. Card 3, step 5 and the `pullOwners` signature (`pullOwners(hubPath string) hubOwners`): no branch for a failed `ReadOwners`.
   - A hub whose `board.json` does not read leaves the collision check with no stated outcome: refuse, skip, or report.
   - Evidence: `ownerset.go:295-308` (`ReadOwners` returns an error on a read or decode failure).
5. LOW, consistency. Card 3, step 3 and the `rewriteFile(file, decoded bool, …)` signature: no branch for a file that exists but does not read.
   - `ReadFile` returns `(File{}, false, err)` for a read error, as distinct from a decode error.
   - Step 3 promises to repair a bad file, but a read error, such as a directory or a permission failure, is neither "decodes" nor "does not decode". `showName`'s `FilePath`-error case is likewise unspecified.
   - Evidence: `internal/machinename/machinename.go:55-68`.
6. LOW, consistency. Card 3, step 6: "for a repair back to a lost name it names writing `{"name": "<name>"}`" makes the way forward conditional on a state `checkCollision` cannot see.
   - Its signature `(name, mine, hubs)` carries no flag that the file failed to decode. The refusal must therefore always name both ways forward, or the condition is unreachable as written. No row asserts the hand-write way forward.
7. LOW, design. Card 2, `writeOwners` bullets: the overlap clauses for `Replace` with `req.Prime` or `req.Slugs` guard a state no caller produces, since card 4 passes `Machine` and `Replace` only.
   - Taken as written, the clauses would leave a stale `lost[...]` entry. The prime branch at `ownerset.go:178-184` records `lost` before the replace selection writes the same subject, so `SetOwners` could report it as both recorded and lost.
   - No row covers either clause.

Tool calls:
- `internal/state/state.go` `WriteJSON`, and the start of `fabricengine` `Bolt.PushRecorded`: to confirm the file write creates the config directory, and whether a push with nothing recorded still pushes an ahead branch. Neither body was in context.
- `pushClassified` and the `BoltSkipFetchFailed` sites: to confirm the ahead branch is pushed with nothing written, and that a fetch failure is a skip rather than a commit. This decides whether the `pushed` report is reachable.
