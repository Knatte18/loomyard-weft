```yaml
lens: fix-affected-set
outcome: reported
```

Rerunning each set-premise finding's search (F1, F2, F3) to check the fix covers every member.

Unchecked: none. F1's set is the one refusal site in `Resume`, which the fix covered, and F2 and F3 were each searched again.

**[LOW] Card 21 adds `handedOver` but doesn't list `Run`, the function that builds the daemon's per-run maps.**
- Changed text: card 21, "The daemon gains `catchUpOwed`, `catchUpNext`, `catchUpDelay` and `handedOver`" and the `handedOver` per-run memory sentence.
- Evidence: the daemon's per-run maps are created in `Run` (`internal/dobbyengine/loop.go:208`: `childSeen: map[string]time.Time{}, wasCovered: map[string]bool{}, liftPending: map[string]bool{}`).
- Card 21 lists `#daemon` but not `internal/dobbyengine#Run` under `**Edit:**`.
- A `handedOver` map set up the same way is a forced edit to an unlisted symbol.
- Setting the map up lazily in `drivePasses` or `catchUp`, both listed, would avoid it, but the card doesn't say so. Either way, the claim that the fix adds no target is unproven.

**[LOW] Card 23 keeps `startHarness`'s test-local field names, but those fields' comment still describes an owner pause.**
- Changed text: card 23, "`startHarness`'s test-local fields keep their names".
- Evidence: `internal/orchcli/start_test.go:162` reads "pauseOwned, when set, wires the owner pause to fail with pauseOwnedErr…".
- That comment sits in the `startHarness` struct, which isn't under card 23's `**Edit:**`; only `startHarness.runWith` and `TestStart_ReconcilesTheHubFirst` are.
- After the change the comment describes a pause that no longer exists, and no listed symbol covers rewording it.

The rest of F2's set is covered:
- Every non-test `pauseOwned`, `OwnerPauses`, "pauses it again" and "was not paused" site in orchcli is under card 23's `**Edit:**`: `cli.go` (`orchCLI`, `wireCLI`), `start.go` (`pauseOwnedRuns`) and `doc.go`.
- The two test sites at `start_test.go:1298` and `:1355` are inside `TestStart_ReconcilesTheHubFirst`, which card 23 lists.
- The loomcli prepush and hibernateengine matches belong to other parts of card 23, outside F2.

F3's predicate set is covered: `DobbyPairSlugs` is defined only in card 20, which the fix edited, and card 21's uses agree with it.

Tool calls:
1. Grep of the repo for owner-pause sites, plus the test function around `start_test.go:1298` and `:1355`, to rerun F2's set; my context held only the narrower orchcli and beamcli grep.
2. Grep for where the dobby keeps and clears its per-run memory, and for every plan mention of `DobbyPairSlugs` and "stands here", to rerun F3's set; neither was in my context.
3. Lookup of the function that builds the daemon's maps at `loop.go:208`, and whether card 21 or 23 lists `Run`; needed to judge the `handedOver` target.
