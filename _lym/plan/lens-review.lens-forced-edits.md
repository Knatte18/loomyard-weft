```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. Findings:

1. **BLOCKING** (Card 9, Card 8): Card 9 deletes `internal/loomcli#spawnStatusPush`, but `internal/loomcli#TestCommitStatusSeam_Real` still swaps it out (`internal/loomcli/wiring_commitstatus_integration_test.go:119-124`). Card 8 lists that test, but its Intent says loomcli's `spawnStatusPush` var "stay[s] for … loom's pull-request record until card 9". So card 8 keeps the swap, and card 9 does not list the test. The build breaks at card 9. The test's comment at `wiring_commitstatus_integration_test.go:104` ("The seam spawns its push instead of running it, so the test swaps … spawnStatusPush") also goes stale at card 8, and no card names it.
   - **Fix:** card 8 drops the swap and the comment from `TestCommitStatusSeam_Real`, since the seam no longer spawns; or card 9 lists the test under Edit.

2. **BLOCKING** (Card 11, `landingshed.Deps`): card 11 adds four func fields to `internal/landingshed#Deps`: `OwnedElsewhere`, `HandOver`, `PushRecords` and `ArchiveParentMerge`. Only `internal/loomcli#landingDeps` is listed as filling them.
   - `internal/testkit/envkit#LandingDeps` (`internal/testkit/envkit/envkit.go:183`) also builds a `landingshed.Deps` for the recipe and seam tests. So does its nil-allowed field map (`envkit.go:75`, `"Landing.MarkTaskDone": true`).
   - The `//lym:seam internal/landingshed#Deps` tests reflect over that struct: `internal/loomcli/landingdeps_test.go:45`, `wiring_seams_test.go:152`, `wiring_beam_test.go:38`, `wiring_darn_test.go:53` and `:415`, `internal/battencli/wire_seams_test.go:208`, and `internal/testkit/envkit/envkit_test.go:36`, `:111` and `:159`.
   - None of these is on card 11. A nil new seam then fails the NilSeams checks, or panics when `landUnderLock` calls it.
   - **Fix:** list `internal/testkit/envkit#LandingDeps`, the allowlist's enclosing member, and the seam tests. State whether the new fields may be nil, for example a nil `OwnedElsewhere` read as owned here, and add matching `internal/landingshed#NewFinalize` nil checks (`finalize.go:106-122`) if they may not.

3. **MEDIUM** (D1 "every detached status push that carries a run transition retires"): `internal/loomcli/pawlwiring.go:110` wires `SpawnPush: func() error { return fabricengine.SpawnPairPush(location) }` into pawl's deps. No card lists it, and the overview's Shared Decisions do not name it as one of the detached pushes that stay. If pawl's push carries the beam run's transitions, it is a detached transition push D1 retires. The plan should either move it onto `pairteardown.PushRun`, or state why it stays detached in `## Shared Decisions`.

4. **LOW** (Card 8): the file header of `internal/battencli/commitstatus.go:6-8` says "The push is spawned, not run: the seam starts fabric's detached push and returns… the next transition's spawn catches the branch up". It is false after card 8. It sits outside every member card 8 lists, so the card should name the file comment, either as the file's self glyph or in its Intent.

5. **LOW** (Card 4, Card 5): the `internal/fabricengine#CoalescePushBothAt` doc (`coalesce.go:80`) says "so it also respawns a side another pusher left flagged". That goes false when card 4 retires the respawn. Card 5 edits the member only for the names, and neither card's Intent rewrites that sentence.

6. **LOW** (Card 14, PATTERN-daemon-wakeups): card 14 adds the statuscommit owed-push retry to the background prose. No card adds `internal/statuscommit/statuscommit.go` to `unboundedWaitFiles` in `cmd/lym/daemonwakeups_test.go:26`, which is the declared list the guard scans for sub-second literals. The same applies to card 3's `acquireTeardownShared` poll if it is meant to sit under the rule. Without the entry, the guard never scans the new unbounded wait the background claims it covers.

Calls made beyond context:
- Grep of the callers of the renamed and deleted members: `PushOwedOffline`, loomcli/battencli `spawnStatusPush` and `guardedSpawn`, and the `DetachedPush`/`CoalescePush*` callers and respawn users. My context had only production callers from the subagents.
- `lym quarry resolve` on the plan's listed glyphs. Context had no resolution for most of them, and the output was too noisy to use.
- `lym quarry resolve` again with status filtering. All listed glyphs resolved. I also grepped the `PushPairAnchored` test callers and the docs naming the spawned transition push.
- Grep of the refusal-text assertions, the failure-class enumerations, the package docs' spawn wording, the daemon-wakeups list and the `//lym:seam` markers on the structs the cards change. These are forced-edit surfaces the pre-read never covered.
- Read of the `landingshed.Deps` builders and the two seam tests' marker blocks, to see who else builds the struct card 11 grows.
- Grep of the `NewFinalize` nil checks and `envkit.LandingDeps`' fills and allowlist, to confirm the envkit builder must change.
