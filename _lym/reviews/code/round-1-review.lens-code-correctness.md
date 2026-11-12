```yaml
lens: code-correctness
outcome: reported
```

Task: check correctness of the code the diff changes (lens code-correctness).

**Per function or branch: case run, result**
- `MigrateConfig`, empty or comment-only input: `document.Kind` is not `DocumentNode`, so it returns the input unchanged. Correct.
- `MigrateConfig`, `test_map_every: 0` with no `test_map_upkeep`: the key is renamed, its line comment is cleared, the value becomes `!!bool false`, and one line is returned. Correct.
- `MigrateConfig`, `test_map_every: 0` with `test_map_upkeep: true` already set: the pair is removed and the existing value kept. Correct.
- `MigrateConfig`, `test_map_every: 5`: the pair is removed. Correct.
- `MigrateConfig`, a non-mapping root such as `null` or `~`: it returns an `ErrInvalid` error, so a layer holding it now fails the load. I could not confirm whether the merge accepted such a layer before this change, so I rate it low risk.
- `MigrateConfig`, a document without the key: it returns the original bytes. Correct.
- `stackLayers` migrate loop: a failed rewrite is marked invalid, with the layer path in the message. Correct.
- `stackLayers`, per-layer order: a lower layer's `test_map_every: 0` becomes `false`, and a higher layer's explicit `test_map_upkeep: true` wins in the merge. Correct.
- `testMapStep`, `StartTestMap == nil` (a beam's dobby): it returns without touching the other seams. Correct. The hub wiring fills every seam together, so the missing nil checks on `TestMapExitRecord`, `PrimeHead` and `GateSlots` cannot panic there.
- `testMapStep`, child live (empty channel): it returns nil and starts no second pass. Correct.
- `testMapStep`, child exited: the channel is read, the child cleared, the exit noted, and nothing started that tick. Correct.
- `testMapStep`, `test_map_upkeep` false: nothing starts. Correct.
- `testMapStep`, registered pass: nothing starts. Correct.
- `testMapStep`, `StartTestMap` returns an error: the error is wrapped and `testMapChild` stays nil, so the next tick retries. Correct.
- `testMapWorkLeft`, no exit record: it returns "no exit record" without reading HEAD or slots. Correct.
- `testMapWorkLeft`, HEAD moved: it starts regardless of the slot count. Correct per D8.
- `testMapWorkLeft`, same HEAD and `Queued == 0`: it returns empty and does not read slots. Correct.
- `testMapWorkLeft`, same HEAD, `Queued > 0` and fewer than two slots: no start. Correct.
- `testMapWorkLeft`, a failing seam: each read error is wrapped and returned. Correct.
- `noteTestMapExit`, repeated failures: `Fail` logs once per episode, and `testMapExitLogged` gates the log-file line. A clean exit recovers and logs once. A signal-killed pass has exit code -1 and counts as failed. Correct.
- `startTestMap`, the lym binary does not resolve: it returns `nil, err` and starts nothing. Correct.
- `startTestMap`, `cmd.Start` fails: it returns `nil, err`. Correct.
- `startTestMap`, normal path: the buffered channel (capacity 1) gets exactly one `PassExit`, so the goroutine cannot leak. Stdout and stderr share one writer, so one pipe. Correct apart from F1 below.
- `dobbyPassInFlight` with `RoleTestMap` spared: a registered pass is no longer waited for. The prime's `dobbyOwner` is `HubOwner`, `EndSparing` spares only push and sync, so the move's `EndDobbies` ends the pass. Correct (`anchordeps.go:91,258`, `anchor.go:468`).
- `dobbyPassInFlight`, a pass spawned but not yet registered: it is still counted for that short window. Correct.
- `state.go`, the removed `LandingsSinceTestMap` field: no `DisallowUnknownFields` appears in `state.go` or `internal/state`, so an old record still decodes. The next `UpdateState` rewrite drops the key. That fits "readable", though no field exposes the value.
- `hubDobbyDeps` seams: `GateSlots` returns `cfg.Slots` with the load error. `PrimeHead` opens the fabric on each call, which only happens once an exit record exists. Correct.

**Findings**

- **F1, MEDIUM, design.** `internal/beamcli/dobbydeps.go:626-655`.
  - `startTestMap` now gives the detached pass a stdout/stderr pipe whose read end only the dobby process holds. Before, output was discarded.
  - The pass runs in its own process group and is meant to outlive the dobby: the `doc.go` bounds say "a dobby that dies without a stop leaves its pass running until HEAD moves". A dobby that dies, or is re-exec'd (Go opens the pipe fds close-on-exec), closes the read end.
  - The pass's next write to fd 1 or 2 then gets SIGPIPE or EPIPE. `runTestMap` writes progress to `cmd.OutOrStdout()` (`internal/gatecli/testmap.go:125`), and a Go process dies on SIGPIPE from fd 1 or 2.
  - That kills the pass mid-queue and defeats the survive-the-dobby behaviour D8 and `doc.go` describe. A pass killed this way writes no exit record, so the next pass plans from scratch.
  - Fix: capture the output through a file, or an inherited fd that outlives the dobby, rather than a dobby-held pipe. Alternatively, have the pass ignore SIGPIPE and write errors on stdout and stderr under `--upkeep`. Add a test that the pass survives the dobby closing its read end.
  - I did not run it, so the SIGPIPE behaviour is inferred from the code.

- **F2, LOW, scope.** `internal/orchcli/doc.go:48-50`, `internal/orchcli/stop.go:37,50,74`, `internal/orchcli/cli.go:125`.
  - These comments and the error text "end the test-map refresh" still describe the old dobby-started, dobby-registered refresh.
  - They say "the test-map refresh the dobby started" and "its own record and process group" without mentioning that the pass registers itself. `beamdown_test.go` asserts the error substring "end the test-map refresh".
  - The behaviour is still correct (`EndRole` with `RoleTestMap` under `HubOwner`). Update the wording to "upkeep pass" in the same diff, and change the asserted substring with it.

**Tool calls**
1. Grep and read in `state.go`, `internal/state` and `runproc/end.go`: checked for `DisallowUnknownFields` and what `EndSparing` spares. The manifest lacked both.
2. Grep and read in `fabriccli/anchordeps.go`, `anchor.go` and `gatecli/testmap.go`: checked that `EndDobbies` reaches `HubOwner` and that the pass writes to stdout. The manifest lacked both.
3. Grep for `dobbyOwner` and `HubOwner`: confirmed the prime's owner is the hub owner. The manifest lacked the definition.
