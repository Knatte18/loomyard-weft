```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: checking that every sentence the fix added or rewrote holds against the code at HEAD.

Findings:

1. MEDIUM — card 6, the new sentence saying the allowlist "admits only writes into a git-owned path or a worktree minted in the same call, and a reconcile's or an adopt's weft is neither".
   The allowlist does not hold that bound.
   Evidence: `cmd/lyx/uncontainedwrite_test.go:81-88` admits entries that are neither.
   - `originreconcilemoves.go`'s lock directory is admitted because "the path is shedrun's, never caller-derived".
   - `scratchlink.go`'s MkdirAll writes into an existing pair records worktree "built from RecordsWorktreePath".
   The `os.Root` route still holds up, but its reason should rest on the allowlist doc comment's rule for new sites (lines 56-60), not on a claim about which entries exist.

2. MEDIUM — card 6, the Docs sentence: "`unwire.go`'s and `codebinding.go`'s file headers and the unwire help in `Command`'s `Long` text, each of which still says unwire leaves every marker in place".
   `codebinding.go`'s header says nothing about unwire.
   Evidence: `internal/fabricengine/codebinding.go:1-4`, "owns the .lyx-warp warp-URL binding … recorded once on weft:main beside .lyx-anchor".
   Only `unwire.go:5-7` and `fabric.go:422-423` make the unwire claim.

3. LOW — card 15, the new `restoreMovePathsToHead` paragraph: "ownership of the move's pathspecs".
   `destroy.go` has no `pathOwnership` constructor for a set of pathspecs.
   The existing ones are listed at `destroy.go:330-395`: `ownedRegisteredLinkedWorktree`, `ownedUnderGeometryRoot`, `ownedWiredJunction` and so on.
   The card's Create list adds none.
   Containment under the weft fits `ownedUnderGeometryRoot`; the "move's pathspecs" gate is new code the card doesn't name.

These claims checked out:
- The Finalize reach to `FastForwardPair` (`landingshed/finalize.go:400`).
- `fastForwardWeft`'s warn-and-go-on behaviour and the code side still landing (`parenttips.go:258-263`).
- `taskAnchor` has two production callers (`archive.go:173`, `teardown.go:490`) and one test helper (`archive_test.go:83`).
- `archivePairLogs` warns and returns when listing fails (`archive.go:176-180`).
- `busyError` is reached only from `waitQuiet` after a successful quiet probe (`teardown.go:458-468`).
- In production, `driverLockBusyError` follows `driverLockHeld` = `driverLockHeldIn` (`teardown.go:170, 304-312`; `cancel.go:69-74`).
- `dirtinessNA` exists (`destroy.go:526`).
- `root.WriteFile` is the established idiom (`hubworkspace_write.go:39`, go 1.26).
- `createSteps` runs `AddPair` before `SpawnPush` (`create.go:134-145`).
- Card 13's count of seven writers matches its list.

Tool calls:
- One Bash read of `uncontainedwrite_test.go:79-92`, `teardown.go:440-469` and a grep of `finalize.go`. My context lacked the remaining allowlist entries, the `busyError` call site and the Finalize call line.
