```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1** (`Strands()` under the store mutex): the search finds no `Strands()` call inside `s.do` left. The only remaining sites are:
  - `chase.go:345`, the chaser probe, which is outside the lock.
  - `server.go:222`, `sweep.go:114` (`sweepOnce`) and `deliver.go:320`, which are scheduler paths outside the lock.
  - `pool.go:186`, the `readStrands` helper, which the new `listStrands` and `verbs.go:170`, `:582` and `:751` call.

  Both callers of `settleUnansweredLocked` (`chase.go:175` and `sweep.go:237`) pass `listing`, and each reads it just before `s.do`.
- **F2** (notifier wirings): `SetNotifier` has four non-test call sites.
  - `burlercli`, `loomcli` and `webstercli` are the three the new orchcli test covers.
  - `battencli/drive.go:448` is the fourth. It calls `notifyRunParent`, not `NotifyAsker`, so it is outside the review's set and the test's scope.
  - The test table lists only those three paths, so a new runner wiring would not be checked automatically. The review did not ask for that.
- **F3** (removed verb in live docs): no Go or `.md` file outside `_lym` and `.lym` still describes the removed `shuttle send` verb as existing.
  - `cmd/lym/removedverbs_test.go` lists it intentionally.
  - The `docs/research/*` hits are research notes about Codex that predate the removal, so I left them as they are.
  - The `shuttleengine` hits are the unrelated "shuttle sends" prose.

Tool calls:
1. Grep for the `Strands()`, settle/notice, `SetNotifier` and removed-verb sets in one command. My context held only the files the parent read, not these sets.
2. Read `battencli/drive.go:444-454`, `sweep.go:108-118` and the `readStrands` callers. I needed to see which notifier the fourth wiring uses and whether `sweepOnce` runs under the lock.
