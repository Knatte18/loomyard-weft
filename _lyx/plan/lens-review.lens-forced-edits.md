```yaml
lens: forced-edits
outcome: reported
```

Scope: what the plan's changes force outside the members its cards name (forced-edits lens).

- **MEDIUM — Card 11 (D10 origin reconcile): two `BranchState` switches are left out.** Card 11 updates `stopsRemoval` and `stopsEntries` but not the two other switches over origin-reconcile states:
  - `internal/cleanupcli/reconcile.go:90-94` groups states into dirty/gone, busy and rewound/rewritten/diverged.
  - `internal/adoptcli/adopt.go:286-290` has the same grouping, and `internal/adoptcli/adopt.go:311` lists the states that stop an owned-elsewhere slug's owner write.
  - With neither switch updated, an `anchor_stopped` branch falls through. Cleanup and adopt then mis-word or ignore the stop, and adopt can write the owner over a weft stopped across a marker change.
  - `internal/adoptcli/doc.go:22` lists those same states in prose.
  - No card names `internal/cleanupcli#…`, `internal/adoptcli#adopt` for this, or the doc line. Card 19 edits `adopt`, but only for the move-record refusal.
- **MEDIUM — Card 9 (D10 push): `ErrAnchorChange` has no failure class.** Card 9 says `pushClassified` "records it as the push's failure". But `pushJob.classifyFailure` (`internal/fabricengine/pushclassified.go`) has no case for it, so it reaches `default: return err, false`. That makes it an unclassified failure (`HasUnclassifiedFailure`), not a failed-push record carrying the `--set` way forward. The same applies to card 13's move-record push stop.
  - Whichever is meant — a new case in `classifyFailure`, or a new `PushFailureClass` beside `PushFailureNotFastForward` and the others in `failedpush.go:26-36` with its `FailedPushWayForward` text — no card lists `internal/fabricengine#pushJob.classifyFailure`.
- **MEDIUM — Cards 16 and 19: new seams that existing code would call as nil.**
  - `dobbyengine.StartDeps` is also built by the test fake at `internal/dobbyengine/start_test.go:32`; card 16 lists `fakeStart.deps`, so that one is covered. But `ensureOne` calls `MoveInProgress` unconditionally, and card 16 does not say what a nil seam means. `hubgeom.DobbyStart` (`dobbygeom.go:56`) is the only production filler.
  - `adoptDeps` is built only at `internal/adoptcli/deps.go:59` and its tests. Card 19 does not say what a nil `MoveInProgress` means for the `TestAdopt` rows that do not set it.
  - Each card needs to state the nil meaning, or the fakes must fill the seam.
- **LOW — Card 18 / Shared Decisions: `ResolveWithAnchor`'s docs go stale.** Card 18 adds a third production caller (`Add`'s new pair at an anchor). Two docs still say "both its callers": `internal/lyxcwd/lyxcwd.go:105` (`ResolveWithAnchor`'s doc) and `docs/shared-libs/lyxcwd.md:48`. Neither card lists them.
- **LOW — Card 1 (D1 import set): two more statements of lyxcwd's import cap.**
  - The file comment of `internal/lyxcwd/leaf_enforcement_test.go` says production code imports "ONLY the standard library and internal/dotgit"; card 1 edits the test function but not this stated reason.
  - `pattern/PATTERN-leaf-packages.md` names `internal/dotgit` as a leaf and may state lyxcwd's import set. It is not on card 1; I did not open it to confirm.
- **LOW — Card 2: `pairLocation` caller names unverified.** The callers are `internal/beamcli/dobbydeps.go:342` and `internal/beamcli/dobbywatch.go:34`. The card names their enclosing functions `dobbyRuns` and `dobbyWatchDirs`, which I could not check within the call budget. The validator resolving them suggests they are right.
- **LOW — Card 17: no role-guard registration needed.** `internal/fabriccli/roleguard.go` lists only `shortname` among fabric verbs, so `anchor` needs no entry there, and no shellguard row names fabric verbs.

Calls made:
1. Grepped `fabriccli/roleguard.go` and `shellguard` for fabric verb registration; my context lacked any guard table.
2. Grepped the `StartDeps`, `adoptDeps`, `promoteDeps` and `createDeps` literals and the `pairLocation` and `create` callers; my context lacked the constructor sites.
3. Grepped `pattern/` and `docs` for lyxcwd's import set and `ResolveWithAnchor`'s caller count; my context had only the background file.
4. Read `pushJob.classifyFailure` and the `PushFailureClass` constants; my context lacked them.
5. Grepped for other lists of origin branch states; only two CLI switches were known.
6. Grepped `BranchState` cases in `adoptcli/adopt.go` and `cleanupcli/reconcile.go`, to confirm the switches the fifth call found.
