```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims, bounds and guarantees, looking for an input or interleaving that breaks each one.

**Claims attacked**

- D1, the revision is taken over the stored entry before the undiscussed copy, and the projection's decision name follows the copy: held. `withMissingRecordsUndiscussed` runs inside `RenderToDisk` (render.go:44), so the revision has to be taken before it, which is what the record says.
- D2, rewriting identical bytes makes no git change: held. Git tracks only the exec bit, and `RenderToDisk` writes with mode 0600 (render.go:56).
- D2/D9, at `http` the CSP blocks the `board-data.js` tag; at `file://` there is no `&lt;meta&gt;` CSP: held.
- D3, the owner-set write and `FabricReplay.Claims` carry the new names: held as stated. The mixed-version bound matches fabric's plain cherry-pick of unclaimed paths (fabricengine/doc.go:1882-1893).
- D3, "a downgrade removes both files": held as bounded. Removal follows the machine-local, gitignored manifest (render.go:32-34, 58-63), and the record's "stale until a newer one writes" covers a machine that never upgraded.
- D4, link scheme filter after HTML escape: held. An escaped `&amp;` can't form an entity, and tabs, newlines and controls are stripped first.
- D5, a free lock means no server, and concurrent opens find the winner: held. `serve` waits for the lock, and the loser exits with success (creelengine/server.go:77).
- D5, `open` takes a state only once it answers a token-carrying request and its pid is live: held against pid reuse and the drain window.
- D5, "the log opens with the server's pid": broken (finding 2).
- D5, "when the prime does not resolve, open returns the error `lymcwd.NamedWorktree` gives": broken (finding 3).
- D5 bound, "the operator's call, with no strand name, waits until the opener exits": broken in one case (finding 4).
- D6, a git-tracked `tasks.json` is skipped by seeding today: held (ideengine/spawn.go:246-255).
- D6, every caller of the regeneration writes the members: held. Every `RegenerateHubWorkspace` caller is in the lym binary, and `cmd/lym/main.go:174` already installs the folder namer there.
- D8, the page's revision and the check's revision cover the same post-migration bytes: held. Both migrations run before `fn` in `boardCriticalSection` (board.go:113-115).
- D8, a status race on release, draft or undraft is caught: held, because the revision covers status.
- D8 bound, sibling side-effect writes are unchecked: held as stated.
- D9, Host check, bearer token, JSON content type plus Origin, no CORS, token kept off the command line and out of the referrer: held.
- D10, "the page shows other machines' writes once they reach this Bolt": broken (finding 1).
- D10, release, draft and undraft refusals are race-free: held through D8's revision.
- D10 bound, the page runs the same checks as the CLI verb: held. `boardcli` has no verb-level refusals outside `boardengine`, and the pre-run closures come back through "resolves its worktree as the other writing board verbs do".
- D11, the `.lym` directory is never staged by the sync: held. `commitWeftAt` seeds the `.lym` exclude before `StageAllAndCommit` (fabricengine/recordsgit.go:418-432).
- D11, a takeover ends the server: held, because the new role stays outside `subjectPassSpared` (runproc/end.go:100).
- D11, the drain completes an in-flight write while a new server starts: held. Both writers serialise on the board flock (board.go:98-104).
- D11, the binary-replacement exit uses creel's watcher: held (creelcli/cli.go:1056).
- D12, the run-agent rule: held (taskguard.go:14-17).

**Findings**

1. MEDIUM, design. `_lym/discussion/decision-record.md:282-284` (D10 `GET /api/board`) and `:329-330` (D11 live view).
   - Summary: nothing ties the board revision the server returns to the bytes it built the projection from.
   - Counterexample: the server loads the store, and a sync pull or another writer then saves `board.json`. The server then hashes the new bytes and returns the old projection with the new revision. The page's next poll (`after=` that revision) waits for a further change, so the page shows a stale board until some later write. That breaks D11's "the page shows other machines' writes once they reach this Bolt".
   - Fix: state that the projection and the board revision are computed from one read of `board.json`'s bytes. The watch loop's comparison uses the same hash.
   - Evidence: decision-record.md:282-284, 329-330. `boardCriticalSection` saves under the flock (board.go:98-104), and the read path takes no lock.

2. MEDIUM, design. `_lym/discussion/decision-record.md:149-150` (D5), `:319` and `:324-325` (D11).
   - Summary: "The log opens with the server's pid" breaks once a second server is started into the same log.
   - Counterexample: S1 hangs while holding the lock. A concurrent `open` that found the lock free a moment earlier starts S2, whose stderr is redirected into the one server log. If the log is opened with truncation, S2's start wipes S1's pid line. If it is appended, the log opens with an older server's pid. S2 never takes the lock, so it never logs a pid of its own. In both cases the next `open`'s refusal points the operator at a missing or wrong pid, and ending that pid can hit an unrelated process.
   - Fix: say whether the log is truncated or appended, and give the hung server's pid from a source only the lock holder writes. Either the refusal prints it, or the lock holder writes its pid into its own file right after taking the lock.
   - Evidence: decision-record.md:149-150, 319, 324-325.

3. LOW, consistency. `_lym/discussion/decision-record.md:146-148` (D5).
   - Summary: "When the prime does not resolve, open returns the error `lymcwd.NamedWorktree` gives and starts nothing" misstates the cited code.
   - Counterexample: `NamedWorktree` "needs neither the worktree nor its records sibling to exist" and only fails reading an anchor. With the prime's worktree missing, it returns a Location whose anchor does not exist, and the spawn then fails on chdir. The failure comes from `fabricengine.PrimeName` or the spawn, never from `NamedWorktree`.
   - Fix: name the error `fabricengine.PrimeName` gives, or have `open` check that the prime's anchored directory exists before it spawns.
   - Evidence: internal/lymcwd/lymcwd.go:97-113, internal/fabricengine/worktreelist.go:178.

4. LOW, design. `_lym/discussion/decision-record.md:162-165` (D5 bound).
   - Summary: "The operator's call, with no strand name, waits until the opener exits" does not hold for a VS Code window that an orch session launched.
   - Counterexample: `vscode.Launch` uses `exec.Command("code", …)` with the environment inherited. The only caller is `machinecli` (cli.go:113), and nothing strips `LYM_STRAND_NAME` there; other spawn sites do strip it (verifytree/verify.go:409, cardverify/runner.go:71). An orch that opens the workspace passes its strand name into every task the window runs. The operator's `alt+shift+9` press is then judged as the orch's call and cut off at the keep-warm cap. If a run agent's name were inherited, D12 would refuse every press in that window.
   - Fix: state the bound: a window inherits the strand name of the process that launched VS Code, and `open` judges the press by it. Alternatively, have the launch strip `LYM_STRAND_NAME` as `verifytree` does.
   - Evidence: internal/vscode/launch_linux.go:19, internal/machinecli/cli.go:113, internal/agentname/agentname.go:59-60.

**Tool calls**

- Grepped `strandNameFromEnv`, the VS Code launch env, `RegenerateHubWorkspace` and namer-install callers, the tracked-`tasks.json` skip, the `.lym` staging exclude and the `boardcli` verb files. My context lacked the hops one past D6, D11 and D12's claims.
- Grepped the `vscode.Launch` callers and every `StrandNameEnv` scrub site. My context lacked who launches VS Code and with what environment.
