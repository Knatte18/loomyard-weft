```yaml
lens: affected-set
outcome: reported
```

**Per change: the affected set found, and the cards that cover it**

1. **Loom `commit-records` envelope** (card 7: `push_skipped`/`push_stopped` go, `pusher_started` added) — readers: `internal/loomcli/commitrecords.go`, `commitrecords_test.go`, `internal/loomcli/doc.go` (card 7), `docs/overview.md:1344` `commit-records` line (card 11). `push_skipped` hits in `internal/fabriccli/*` and `fabricengine/recordsgit.go` are another verb's envelope, not affected. No stencil/spec under `contracts/` or `plugins/` reads these keys. Fully covered.
2. **"A transition pushes synchronously" behaviour/claim** (cards 7, 11) — readers: statuscommit package doc + battencli/loomcli seam docs (card 7); `pairteardown/doc.go:168` "PushRun is the run's synchronous pair push" (card 4); `PATTERN.md`, `pattern/PATTERN-run-fsm.md`, `PATTERN-pair-remote.md`, `PATTERN-daemon-wakeups.md` (card 11); `docs/overview.md:339` statuscommit line (card 11); `internal/fabricengine/pushanchored.go:23` `StatusPushLockWait` doc (**not covered**); `internal/tenderengine/report.go:114` (**not covered**); `internal/tendercli/cli.go:215` `lym tender resume` help (**not covered**). `docs/research/*` are historical records (out of scope). Partly covered.
3. **Batten marker `push_pending`** (card 7) — readers: `commitstatus.go`, `commitstatus_test.go`. Covered by card 7.
4. **`statuscommit.Deps` loses `Push`/`Admit`/`Owed`/`Sleep`** (card 7) — production constructors: `battencli/commitstatus.go:243,252`, `loomcli/wiring.go:346`. All in card 7. `fakeGate` in `internal/shedengine` tests is a different type. Covered.
5. **Loom `commitStatusDeps` type loses push fields** (card 7) — only production constructor `commitStatusDepsGuardedBy` (`loomcli/wiring.go:220`), also feeding `loomCLI.commitStatusDeps` (`wiring.go:264`, which only overrides `Finish`/`SetBoardStatus`). Card 7. Covered.
6. **Batten `commitStatusDeps` keeps `Push`** — out-of-scope users `readopt.go:95-99`, `cancelcmd.go:308,490-498` are kept deliberately (card 7); `drive.go:298,349,358` reach `commitRunRecords` (card 7). Covered.
7. **Worktree-Teardown now pushes the pair first** (card 10) — `TestBattenIntegration_Rows` drives the real wire through Worktree-Teardown and other sub-tests assert the records branch on the remote (`lifecycle_integration_test.go:660-730`); the push lands before removal, so those assertions survive. Card 10 adds a row. Covered.
8. **Run read leaves `PushOwed`/`CodeTipMissing` unset** (cards 1, 9) — readers `runfsm`, `DobbyRuns`; card 9 states the code-tip settle change. Covered.
9. **Pause → no push after lift** (D7: pusher exits on a pause) — `lym tender resume` spawns a push only for a pair with a deferred-push pending flag (`tenderengine/resume.go:309`, `tendercli/deps.go:142`), so the restart after a lift falls to the dobby tick (card 8). Behaviour fine; the tender docs describing it are the gap in change 2.

**Findings**

- **MEDIUM · consistency · card 11 (`11-pattern-and-docs.md`)** — The plan's doc sweep misses the tender help and engine doc, which state the rule this plan makes false: `internal/tendercli/cli.go:215-217` (the user-facing `lym tender resume` help: "A run's transition pushes synchronously, so no pair push is spawned for one.") and `internal/tenderengine/report.go:114` (`PushDeferred` field doc: "A run's transition pushes synchronously and owes no lift a push."). After D7 a pause leaves a pair's commits unpushed until the dobby tick restarts its pusher after the lift; neither text says so. Fix: add both to a card that edits prose — they are Go code, so an Edit card (e.g. card 7 or card 5), not card 11's Prosa list — rewording them to: a lift spawns no push for a transition's commits, since the dobby's tick restarts the pair's pusher.
- **LOW · consistency · card 1 (`01-fabric-owed-unrecorded.md`)** — `fabricengine.StatusPushLockWait`'s doc (`internal/fabricengine/pushanchored.go:23`) says it is "the bounded push-lock wait that `commit-records` and a run's transition push pass"; after cards 6 and 7 neither pushes, and the pusher and teardown pass it. No card lists it. Fix: add `internal/fabricengine#StatusPushLockWait` to card 1's Edit with an Intent line rewording the doc to the pusher and Worktree-Teardown.

Tool calls:
1. Grep for readers of `push_skipped`/`push_stopped`, `commit-records` push wording in docs/stencils, and "synchronous" transition-push docs — manifest lacked readers outside the read files.
2. Grep tender engine/CLI, runfsm/statuscommit/battencli/pairteardown/battenshed package docs and docs/overview.md for the wording — to read the hits call 1 surfaced.
3. Read `tenderengine/report.go` 105-122, `tendercli/cli.go` 205-225 and `PushDeferred` callers — the tender lift was one hop beyond the manifest.
4. List integration tests that read origin after a run — to find tests relying on the transition push.
5. Grep those batten integration tests for their origin assertions.
6. Read `lifecycle_integration_test.go` 660-730 — to judge whether the teardown rows' remote assertions survive card 10.
