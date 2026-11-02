```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens for the hibernate wake/cancel/notice diff.

## Per change: the set found, covered or not

1. **`Deps.NoticePath` added.**
   - Set: every `hibernateengine.Deps` construction. A grep for `NoticePath` and `Deps{` finds only `hubDeps` (`internal/hibernatecli/deps.go:117`) and `fakeWorld.deps` (`world_test.go:461`). **Covered.**
   - Set: the notice file's location, `HibernateDirName` under `lymdirs`. `deps.go` builds it from `HubScratchDir` and `lymdirs.HibernateDirName` and declares no raw `.lym` path. The `lymdirs` comment and the `hibernatecli` doc were updated. **Covered**, and consistent with `PATTERN-durable-vs-ephemeral-state`.
   - Set: the `PATTERN-daemon-wakeups` entry for `watch.go`. The retry loop starts at one second, doubles to thirty and has a ten-minute deadline. It uses no sub-second literal and no ticker. **No violation.**

2. **`Cancel` on a terminal request now lifts the owned entry.**
   - Set: `cancelCmd`'s `Long` text and `Cancel`'s doc comment. Both were updated, the former awkwardly (see Finding F-A).
   - Set: `internal/hibernatecli/cli.go:101` `Short`. It still reads "withdraw the pending hibernate request and lift the pause it added". It stays true for a terminal request but says only "pending".
   - Set: `contracts/orchrole/orch-role.md:264-265`. It says `cancel` "withdraws" the request and does not say a done request is out of scope, so it is not falsified. **Covered.**
   - Set: `fabriccli/anchor.go:328` and `anchordeps.go:46` read the request file only for daemon liveness. A terminal `Cancel` rewrites nothing and kills nothing, so they are unaffected.
   - Set: the `PATTERN-adopt-hold` and `PATTERN-refusal-way-forward` entries. The refusal text is unchanged. **Covered.**

3. **The stop on a shared branch now lifts the hub entry and holds the stopped runs.**
   - Set: `docs/overview.md:481` and `docs/overview.md:495`, and `contracts/orchrole/orch-role.md:264`. None describes the old "stays live" behaviour. A repo-wide grep for "keeps the hub entry live" or "stays live" outside the engine finds no stale claim in docs or stencils. **Covered.**
   - Set: the `hibernateengine/doc.go` paragraph. It is updated, but see F-B.

4. **The notice is a short line plus `notice.txt`, with a retry.**
   - Set: `orchcli/doc.go:150`. It says only that hibernate keeps `NotifyPrime`. `NotifyPrime`'s doc says "the notice is not retried", and `hubDeps` still calls it once per attempt. The retry lives in the engine, so the doc stays true.
   - Set: the board entry's second Expected bullet, "reaches the orch by its real creel name". No change to `NotifyPrime`, `PrimeOrchStrand` or `OrchStrandOf` is in the diff, and the diff only retries. This is the handler's focus point 1. From my lens, the resolution path (`orchroute.StrandOf`) is untouched, so nothing in the affected set was changed to meet that bullet. I report it as a scope gap (F-C) rather than rule on it.

## Findings

- **F-A (LOW, consistency):** `internal/hibernatecli/cli.go:101-108`. `Short` still says "pending", and the `Long` edit leaves line 106 as one long run-on line. The text is correct, but `Long` was not re-wrapped like the surrounding lines. Fix: re-wrap `Long` and say "pending or ended request" in `Short`.
- **F-B (LOW, consistency):** `internal/hibernateengine/doc.go` still ends the Cancel paragraph with the "Bound: a daemon killed between…" sentence, now placed after the new terminal-request sentence. The terminal-request sentence sits between the bound and the earlier cancel flow, so the bound reads as describing the terminal case. Fix: move the new sentence above the `Bound:` line. I read this from the diff only, not the file.
- **F-C (MEDIUM, scope):** the second Expected bullet ("reaches the orch by its real creel name") has no change in the diff. `hubDeps`'s Notify still resolves through `orchcli.NotifyPrime`, and there is no test for the resolution. `ly:orch` is a fixed fallback name in `OrchStrandOf` when reed no longer holds the GUID. The diff only retries for ten minutes. Fix: either show in the diff how the real name is resolved, or record a decision that the retry is the answer.

No missing PATTERN declarations are in the diff, and the directive's declared-changes list is not in my context, so I raise nothing under "Declared PATTERN changes landed". The commit message carries no `Pattern-Departure:` trailer, and the diff needs none.

## Tool calls
1. Grep for `Deps{`, "stays live", `hibernate cancel` and `HibernateDirName` across the repo. My context lacked other consumers and docs.
2. Grep of `cli.go` and `cli_integration_test.go` for Cancel text. My context lacked the integration test's assertions; it has no cancel assertion.
3. Read of `cli.go:95-165`. My context lacked `cancelCmd`'s `Short` and surrounding lines.
