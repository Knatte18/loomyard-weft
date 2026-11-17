```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break every bound, guard and claim about existing code in the record.

**Claims attacked**
- D1, Go's default JSON HTML escaping keeps `</script>` from ending the script: held.
- D2, rewriting identical bytes makes no git change: held.
- D2, the page loads data only through `<script src>` because `file://` blocks `fetch`: held.
- D3, `RenderToDisk` writes both files and its manifest removes stale ones: held (render.go:40-60).
- D3, the sync stages the whole Bolt tree: held (`commitDirty`, boardengine/sync.go:59-61).
- D3, the owner-set path commits every name `Render` returns: held (ownerset.go:286-321, 130-155).
- D3, `FabricReplay.Claims` makes a replay re-render the new files "instead of dropping them": broken, both in what it says about unclaimed paths and in its mixed-version bound (F1).
- D3, a commit that changes only rendered files replays as empty: held (replay.go:462).
- D4, the link allowlist "http, https or a relative path": broken (F3).
- D5, "runs from anywhere in a worktree" through `lymcwd.ResolveWithin`, "as the other board verbs do": broken (F2).
- D5, `--board-path` points the verb only at that directory's `board.html`: held. The flag is hidden and its pre-run step skips resolution (boardcli/cli.go:118, 134-136).
- D5, `lym board rerender` renders and syncs, so the file then exists: held for the file. "Committed" fails when the board config sets SkipGit/SkipPush; this is minor and not filed.
- D5, `xdg-open` with no desktop runs the browser in the foreground: held, the bound is stated.
- D6, one `beam start` per multi-root window: held. Folder `tasks.json` never carries it.
- D6, `lym beam start root` names the prime: held (beamcli/cli.go:457, target.go:16).
- D6, the bound listing the windows where a key names a missing task: broken in its key attribution (F4).
- D6, every copy of `board open` gives the same result: held.
- D6, every seed or spawn rewrites the keys with the block: held. A binary older than the change rewrites the block without them (F5).
- D7, "the README already wraps each project in `<details>`": held (render.go:4).

**F1 — MEDIUM, design. D3 bound and its claim about replaying unclaimed paths.**
A replay sends a commit through the by-entry merge only when a registered replayer claims every path it touches. Any unclaimed path turns the whole commit into a plain cherry-pick (fabricengine/replay.go:742-763; `cherryPickOnto` reports dropped commits).
Counterexample: machine A runs the new lym, machine B an older one. A's board write commits `board.json`, `README.md`, `board-data.js` and `board.html`. B's older `Claims` knows neither new file. When B replays its own pending board commit after a sync, A's commits on origin carry paths B cannot claim, so B's by-entry handling of commits holding the new files falls back to a cherry-pick. That is a conflict-prone whole-commit pick, not a stale file.
D3's bound says older binaries only leave `board-data.js` stale and that each case heals on the next write. It never names this cherry-pick fallback for commits that carry the new files. It also says unclaimed paths are "dropped", which is not the mechanism.

**F2 — MEDIUM, design. D5's cwd resolution contradicts how the board subtree resolves its cwd.**
The board subtree's `PersistentPreRunE` runs the strict `lymcwd.Resolve(cwd)` for every subcommand except `board` and `intake`, and aborts on error (boardcli/cli.go:125-152). It does this before any verb's `RunE`.
Counterexample: `lym board open` run from `<anchor>/internal` is refused by the pre-run hook, so the verb's own `ResolveWithin` never runs. The D5 criterion "run from a subdirectory of a worktree … hands the opener seam the absolute path" fails.
D5 states both "resolves its cwd with `lymcwd.ResolveWithin`" and "reads the board directory as the other board verbs do", which cannot both hold. The record must say the pre-run hook exempts or relaxes `open`.

**F3 — MEDIUM, design. D4 leaves "relative path" undefined in the link allowlist.**
D4 keeps a link when its target is `http`, `https` "or a relative path" and does not define relative. Counterexamples a naive "not http(s)://" or "no ://" test lets through:
- `javascript:alert(1)` and `data:text/html,…` contain no `://`.
- `JavaScript:` passes a case-sensitive scheme test.
- `//host/share/x` is scheme-relative. On a page opened at `file://` it resolves to `file://host/share/x`, a UNC/SMB fetch on Windows.

The bound needs to say: no scheme, compared case-insensitively after trimming whitespace and control characters, and not starting with `//` or `\\`. Evidence: decision-record.md:82-83.

**F4 — LOW, consistency. D6's bound pins the git-tracked `tasks.json` case on the wrong key.**
Line 122-123 lists "any window whose worktree has a git-tracked `.vscode/tasks.json`" as a case where `alt+shift+0` misses its task. `beam start` lives in the workspace files' `tasks` key, never in `tasks.json`, so skipping a tracked `tasks.json` drops `board open` (`alt+shift+9`) and `reed attach`, not `beam start`. A workspace window over such a worktree still has `beam start`. Evidence: ideengine/spawn.go:246-255; decision-record.md:111, 122-123.

**F5 — LOW, design. D6 has no mixed-version bound, though D3 has one for the same exposure.**
The workspace files are rewritten by every `RegenerateHubWorkspace` caller: fabric verbs through `regenerateAfterVerb`, anchor changes, pawl wiring and ide spawn (hubworkspace.go:130-134; anchorchange.go:539; loomcli/pawlwiring.go:116). The keybindings block is rewritten whole on every seed or spawn (keybindings.go:34-40).
Counterexample: after the upgrade, an older lym on the same hub runs a fabric verb. That rewrites the hub and beam workspace files without the `tasks` key and removes `beam start`. An older `lym ide spawn` rewrites the keybindings block without `alt+shift+0` and `alt+shift+9`. Each binary undoes the other's output on every write.
D6's bound names only "a window seeded before the upgrade until it is seeded again".

**Tool calls**
- Read boardcli/cli.go:120-160 and grepped `claimantOf` in boltreplay.go: the manifest lacked the board pre-run hook body and the replay's handling of unclaimed paths.
- Grepped `claimantOf(` callers across fabricengine: needed the sites in replay.go that the manifest did not cover.
- Read fabricengine/replay.go:600-630 and 730-790: to see what classifying a partly unclaimed commit does.
- Grepped boardengine `Sync` staging and `cherryPickOnto`: to test D3's claim that the sync stages the whole Bolt tree.
