```yaml
lens: row-validity
outcome: reported
```

Row-validity lens: whether each planned test row reaches its branch, would fail if the code were wrong, and is needed.

**MEDIUM — Card 4, `TestName_Set` row `CwdHubIsSweptOnce` cannot reach its branch.**
- In `hubsToSweep`, the hub of the working directory comes from `lymcwd.ResolveWorktree(cwd)` and `preflight.BoardLymPresent`, which the card calls directly rather than through a `deps` seam (Card 4 Intent, `hubsToSweep` paragraph).
- `TestName_Set` is a tier-0 fake test. Its cwd is a temporary directory, which is not a lym worktree, so the resolve fails and the cwd hub is never added.
- The row therefore cannot produce "the cwd lies in a hub, and that hub is also in the registry", and the "once" de-duplication goes untested.
- The tier-1 `TestName_SweepsHubs` holds the cwd in hub B, but B is not recorded, so it does not test "once" either.
- Fix: add a `cwdHub func(cwd string) (string, bool)` seam to `deps`, or move the row to tier 1 with B both recorded and holding the cwd.

**MEDIUM — Card 1, `ReplaceSetSelectsAfterThePull` does not reach the empty-entry criterion.**
- The row passes `Replace {"box", "", "old"}`, but its setup records `box` as owner of `beamer`, the fixture's only unowned beam (`internal/boardengine#newOwnerFixture`).
- No unowned beam is left, so `""` matching an empty `OwnerMachine` would go unnoticed if the code were wrong.
- The prime is `box`, so an empty prime owner is not exercised either.
- The acceptance criterion "An empty `names` entry is neither a former name for D6's replace set…" therefore has no row that fails if the code is wrong.
- Fix: keep one beam unowned in this subtest (for example seed `farbeam` with no owner instead of `far`, or keep `beamer` unowned and move `box` to a new beam), and assert it stays unowned.

**MEDIUM — Card 4, the `readOwners` error branch has no row.**
- Step 4 says "a read error fails that hub as in step 7 and the check goes on". `fakeHubs` can return that error, but no `TestName_Set` row does.
- It is also unstated whether that hub still gets its `setOwners` call. The row should pin the hub's outcome and that the other hubs are swept.

**MEDIUM — Card 4, the refusal for a failed registry read has no reaching row.**
- "A failed registry read refuses before any write, naming the record's path" names no input or test.
- A `hubs` seam that returns an error would reach it in `TestName_Set`. Add a row that asserts no file was written.

**LOW — Card 3, a show with a hostname error and no file has no row.**
- `readCurrent` keeps `""` for the hostname name when `hostName` errors, so with no file the show has no valid name.
- The card does not say what the show prints in that case, and no `TestName_Show` row reaches it.

**LOW — Card 4, nil hub seams have no stated meaning, and `TestName_Show` is not updated.**
- Card 3 says "a hand-built `deps` sets every field". Card 4 adds `hubs`, `pull`, `unpushed`, `readOwners` and `setOwners`, but does not list `internal/machinecli#TestName_Show` under Edit to set them.
- Card 4 also does not say that the show path never calls them.
- The show does not touch those seams, so nothing breaks. The stated invariant is false, though, and a nil seam has no stated meaning.

**LOW — Card 4, `TestName_SweepsHubs` leaves its seeding helpers and subtest isolation unstated.**
- Seeding beams, a done beam and the prime on A and on its `CloneSecond` needs `boardengine.LoadConfig`, `boardengine.New(...).UpsertTask` and `SetOwners` with `Beam` and `Prime`. None of these is under Uses, and the card does not say how the seed is written.
- It is not stated whether each subtest builds its own hubs and file. If `RenameSweepsEveryHub` and `FailedHubIsSweptByTheReRun` shared one fixture, the second would start with the name already set, and its "re-run sweeps B" assertion could pass without testing the failure path.

**LOW — Card 2, `TestFromThisHost` restates its implementation.**
- The row compares `FromThisHost()` with `FromHostname(os.Hostname())`, which is the function's body. It fails only if the function stops calling the hostname rule.
- That makes it a one-symbol test, which the rubric's "Test economy" flags. It is cheap and pins the export, so it is no defect beyond noting.

**LOW — Card 4, tier 0 and tier 1 overlap.**
- `FailedHubIsReportedAndOthersSweep` (tier 0) and `FailedHubIsSweptByTheReRun` (tier 1) both pin "failed hub reported, others swept, exit non-zero". The tier-1 test adds only the re-run.
- `CollisionIsRefusedNamingHubAndOwner` (tier 0) and `CollisionWithAnOriginOnlyOwnerIsRefused` (tier 1) both pin the refusal and that no file is written. The tier-1 test adds only the owner that origin alone held.
- Keep each tier-1 subtest's assertions to the part only it adds.

Rows checked with no defect:
- **Card 1:** `ReplaceSetWithNoMatchingOwnerWritesNothing`. The outcome loop pins that origin's head stays at the seed.
- **Card 3:** every `TestName_Show` row. The locate seam produces each state.
- **Card 4, `TestName_Set`:** `InvalidName`, `NoDirectory`, `RelativeVariable`, `Collision`, `OwnFormerName`, `HostnameName`, `PullSkip`, `FailedHub`, `LostEntry`, `UnpushedBefore`, `NothingRecorded` and `EmptyEntryDrop`.
- **Card 4:** every `TestNextFile` row.

Tool calls:
1. Grepped `isDone`, `StatusDone` and `fabricengine.BoardDir`, because card 1's done-beam seed and card 4's board-directory seams depend on them and my context did not show them.
