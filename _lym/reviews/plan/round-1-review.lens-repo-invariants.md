```yaml
lens: repo-invariants
outcome: reported
```

**Per card: each invariant checked, held or broken**

| Card | Reflective seams | Tier tags | Doc text | Card order |
|---|---|---|---|---|
| 1 | held (`SyncOptions`, `PairPush`, `RunPush` have no reflective test in manifest) | held (new test in external `fabricengine_test`, beside existing tier1 files) | **broken**: `StatusPushLockWait` doc (finding 2) | held |
| 2 | n/a | held | held (`doc.go` extended) | held |
| 3 | n/a | held | held (`doc.go` extended) | held (only own symbols) |
| 4 | n/a | n/a | held (`pairteardown/doc.go` edited) | held (after 1, 3) |
| 5 | n/a | held (`testmain_test.go` untagged, as `cmd/lym/tmuxisolation_test.go:36` requires) | held (`lymdirs/doc.go` extended; `unboundedWaitFiles` gains runpusher file) | held (uses card 2) |
| 6 | n/a | n/a | held (`loomcli/doc.go` edited) | held (uses cards 1, 5) |
| 7 | held (`commitStatusDeps`/`statuscommit.Deps` not reflected over) | n/a | held (statuscommit package comment, `commitstatus.go` file comment, `loomcli/doc.go`, `unboundedWaitFiles` drop all listed) | held (`commitStatusDepsGuardedBy`'s test user `TestPrePushCheckWiredIntoDeps` is edited; `runpusher.Start` from card 5) |
| 8 | held (`startPusher` on `battenCLI`/`loomCLI`, not on `c.env`) | n/a | **broken** (NIT): `beamcli/doc.go` (finding 3) | held |
| 9 | held (`runFakes` edited) | n/a | held | held |
| 10 | **broken**: `TeardownDeps` is reflected over through `shedrecipe.Env.Teardown` (finding 1) | n/a | held (`battenshed/doc.go` edited) | held (uses 1, 2, 5) |
| 11 | n/a | n/a | held (only `PATTERN-run-fsm.md`, `PATTERN-daemon-wakeups.md`, `PATTERN-pair-remote.md` in `pattern/` name `statuscommit`; all listed) | held |

Other doc text checked: `internal/fabricengine/pushowed_test.go:1` header says `IsPushOwed` is "the classification a transition seam retries a failed push by" — after card 7 it is the pusher's (NIT, card 3 edits `TestIsPushOwed` in that file but not the header). `docs/research`/`docs/benchmarks` hits are measurement/history records, exempt.

**Findings**

1. **BLOCKING · consistency** — card 10 (`10-teardown-pushes-first.md`): it adds `Push` and `Level` to `battenshed.TeardownDeps`, both nil-legal, but two reflective seam tests then fail and card 10 lists them only under `Uses`, with no card editing the files that must change: `internal/testkit/envkit/envkit.go:671-674` (full env fills `Teardown` with only `Shutdown`/`Remove`) and `envkit.go:58` `nilLegal` (lists neither `Teardown.Push` nor `Teardown.Level`) → `TestFullEnv_FillsEverySeam` fails (`envkit_test.go:31` already marks `//lym:seam internal/battenshed#TeardownDeps`); and `internal/loomcli/wiring_seams_test.go:96-97` (the loom wire leaves batten `Teardown` seams nil; `intentionallyNil` lists only `Teardown.Shutdown`/`Teardown.Remove`) → loom's `TestWire_EverySeamFilled` fails (`wiring_seams_test.go:182-210`). Fix: add `internal/testkit/envkit#nilLegal` to card 10's `**Edit:**` with `Teardown.Push` and `Teardown.Level` (card 10 documents nil as their default), or fill both in the envkit full-env builder, and add both paths to `internal/loomcli#intentionallyNil`.
2. **LOW · consistency** — card 1 (`01-fabric-owed-unrecorded.md`), `internal/fabricengine#StatusPushLockWait`: its doc (`internal/fabricengine/pushanchored.go:23`) says it is "the bounded push-lock wait that `commit-records` and a run's transition push pass"; after card 7 neither pushes — the pusher and Worktree-Teardown pass it. Card 1 edits that file's push symbols but not this constant.
3. **NIT · consistency** — card 8 (`08-restarts-start-pusher.md`), `internal/beamcli/doc.go`: card 8 makes the prime dobby's `Tick` restart the prime's pusher "before its other work", but the doc's dobby-seam section (`internal/beamcli/doc.go:224-231`, "Only the prime's dobby carries the hub's hibernate request and Slack relay"; "Bound: `Deps.Tick` runs after the tick's other steps") does not say so; card 8 lists it under `Uses`, not `Edit`.

Tool calls:
1. Grep battencli/loomcli wire seam tests and envkit for reflection/nil-legal lists — manifest named these tests but not their contents.
2. Read envkit `nilLegal`, full-env `Teardown` fill, and loom `TestWire_EverySeamFilled` — to confirm how `TeardownDeps` fields are checked.
3. Grep `Teardown.*` entries in `intentionallyNil` and docs naming the old push rule — doc text outside the manifest.
4. Grep `pattern/`, stencils and `doc.go` files for statuscommit/synchronous-push wording — to find docs no card lists.
5. Read `beamcli/doc.go` 188–232 and runproc `RolePush` doc — the dobby-seam doc card 8 touches behaviourally.
6. Head of test files and the tier-tag scan in `cmd/lym` — to confirm the tier-tag convention for card 5's `testmain_test.go`.
