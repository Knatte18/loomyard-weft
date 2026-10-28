```yaml
lens: code-pattern-fit
outcome: reported
```

Task: code-pattern-fit lens. For each rule, I checked the changed files and say whether it held or was broken.

**Rules checked**

- `PATTERN-fabric-git`: held. The hub walk, `upstreamSHAAt` and `List` are in `internal/fabricengine`. The CLI only reuses `acquireBranchLock`, which already read the branch before this diff.
- `PATTERN-parent-lock`: held. The `Pattern-Departure: PATTERN-parent-lock` trailer is on 65d012cc, the commit that adds the pair pull. The named entry does forbid moving a local branch outside a run's Finalize. The decision record requires this trailer. The trailer excuses only that departure.
- Declared PATTERN changes: held. The diff touches no `PATTERN.md` or `pattern/` file, and no declared change or new invariant needs one.
- `PATTERN-pair-parent` and `PATTERN-owner-machine`: held. No parent is picked, and the pull is not a start, resume or landing, so ownership does not gate it.
- `PATTERN-cwd-resolution`: held. `fabricengine/hubpull.go` calls `lymcwd.NamedWorktree`, and other `fabricengine` files already do the same, such as `addanchor.go` and `anchorchange.go`.
- `PATTERN-cliwire-sole-wiring`: held. `runHubPull` takes the location it is handed and resolves nothing itself.
- `PATTERN-cli-cobra`: held. `--hub` is a flag on the existing `pull` subcommand.
- `PATTERN-spawn-observability` and `PATTERN-durable-vs-ephemeral-state`: not applicable. The diff spawns no process and adds no state file.
- `PATTERN-refusal-way-forward`: not applicable, since it covers webster, shed and loom refusals. The hub pull's stops name a way forward anyway, except the lock-acquisition and worktree-list errors.
- `PATTERN-test-isolation`: held. Both test files use `hubforge` fixtures and `gitkit`, and the engine file shares the existing `TestMain`.
- `PATTERN-test-speed`: held except for one item (finding 1). Each file has one `tier1` tag and `t.Parallel`.
- Docs-in-same-commit rule from `CLAUDE.md`: held. Both `doc.go` files carry the new section, and `docs/overview.md` needs no change because no module row changed.
- Anchor-lock refactor: held. A search found no remaining reference to the removed `anchorLockOwnerRunID` or the old log text, and the `anchor` verb keeps the run id `fabric-anchor`.
- Comment conventions: one item broken (finding 2). The rest are on their own lines, with no end-of-line comments and file headers present.

**Findings**

1. LOW, `consistency`, `internal/fabricengine/hubpull_integration_test.go`, `TestBoltPull_WaitsForTheBoardWriteLock` (the `time.After(200 * time.Millisecond)` select).
   - The test relies on a real-time delay.
   - `PATTERN-test-speed` says "injected clocks", and the `testing` skill forbids wall-clock dependence.
   - The comment explains it: a blocked flock gives no signal to wait on. It is a negative assertion, so a slow machine cannot make it fail wrongly.
   - I would take this only if the repo's tests can wait on a signal instead. It is the weakest finding.

2. LOW, `consistency`, `internal/fabricengine/bolt.go` (the `Bolt.Pull` doc comment, "as PullThenCommitWritten reports it").
   - A doc comment must not be stated in terms of another named symbol (`scribe:code-quality`, "Self-contained").
   - The `PullHub` doc comment likewise says "through Pull" and "the pull-first path". The reader there needs the `Fabric.Pull` contract, which that comment names loosely.
   - The fix is to state the skip and detail contract inline.

**Outside my lens**

- `internal/fabriccli/weft_verbs.go` rewrites the existing `pull` help bullets about fast-forward and history rewrite. That change is unrelated to `--hub`. The scope owner should decide whether it is licensed.
- The engine test uses a recording lock seam, so it does not show a pair pull actually waiting while another owner holds the parent lock. The tests lens owns that.

**Tool calls**
1. One Bash grep for `anchorLockOwnerRunID`, the old lock-wait and release log text, and `fabric-anchor`. My context did not cover other users of the constants and strings the diff removed.
2. The same Bash call also grepped `lymcwd.NamedWorktree` in non-test files of `internal/fabricengine`, to confirm that package importing `lymcwd` is established.
3. The same Bash call also grepped `t.TempDir` and `os.TempDir` in `internal/fabricengine` tests, to see whether `t.TempDir` has precedent there.
