```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I looked for the callers, readers and docs it forces to change and checked whether they were updated.

**Per change: set found, covered or not**

- **`RefuseUnlessPrime` replaced by `ResolveBattenPrime` for batten, adopt, cleanup and `lyx beam`.**
  - The only remaining non-test reference is its own definition at `internal/battencli/refusal.go:44`.
  - `lockStore`, `armAt`, the cancel command and the adopt/cleanup wiring all moved to the new guard.
  - Covered.
- **`NotifyPrime` replaced by `NotifyParent` / `ParentOrchRecorded` for a run's pair.**
  - Loom's driver hand-back and `OrchRecorded`, `SetNotifier` and `NotifyParent` now use `NotifyParent`.
  - Only hibernate (`internal/hibernatecli/deps.go:112`) and the push notifier (`internal/orchcli/pushnotifier.go:46`) still call `NotifyPrime`, as D13 requires.
  - Covered.
- **`NearestLiveOrch` consumers.**
  - Batten's notices (`internal/battencli/notices.go:37`), `landingLineReachesPrime` (`internal/battencli/land.go:270`) and `orchcli/notify.go:98` use it.
  - Covered.
- **`ResolveParent` kept for the parent pair itself.**
  - `internal/loomcli/landingdeps.go:471` is the parent-records commit, which D13 keeps on `ResolveParent`.
  - `internal/battencli/land.go:255` uses it only as the walk's start point.
  - Covered.
- **`AddOptions` changed from an alias of `SyncOptions` to its own struct, and `ErrBranchExists` message reworded.**
  - The diff updates `add_branch_exists_test.go`, `origin_integration_test.go` and `slug_test.go`.
  - I did not search for other struct-literal users.
  - Not checked beyond those test files.
- **Docs.**
  - `docs/overview.md` covers orch, cleanup, adopt, the `beam` module and Worktree-Create.
  - `plugins/ly/skills/board/SKILL.md` covers the project kind, `P` handle, `parent_beam` and the `beam` flag.
  - The orch role stencil carries the `parent_beam` filter, and `PATTERN.md` and `pattern/` are in the stat.
  - Covered.
- **Parent name given to spawned roles and to the decision guard.**
  - `ParentName` is still what `ResolveParent` returns, which is the recorded parent (`internal/hubgeom/hubgeom.go:35`, `:97`, `:138`; `webstergeom.go:45`).
  - `lyx loom decision` still resolves the parent through `ResolveParent` (`internal/loomcli/decision.go:751`).
  - The Discussion-Write parent-review reviewer also still uses `ResolveParent` (`internal/loomcli/wiring.go:390`).
  - Not covered; see finding 1.

**Findings**

1. **MEDIUM, class `consistency`:** parent names still follow the recorded parent while notices follow the nearest live orch.
   - D13 says "Notices and parent names go to the nearest live orch up the chain". The diff routes the notices through `NotifyParent`, but `ParentName` for the parent directive, burler and webster prompts stays on `ResolveParent`.
   - So do `lyx loom decision add --by parent` and the Discussion-Write parent-review reviewer.
   - Take a member of a beam whose orch is stopped or never started. Its notices reach the prime's orch, but its agents are told their parent is `&lt;shortname&gt;:&lt;beam&gt;:orch`.
   - The decision guard compares the caller's strand to that recorded name (`internal/loomcli/decision.go:565`), so the prime's orch that received the notice is refused with "the caller is a task session".
   - The parent-review gate addresses a session that does not exist.
   - Evidence: `internal/hubgeom/hubgeom.go:35`, `internal/loomcli/decision.go:751`, `internal/loomcli/wiring.go:390`, `internal/hubgeom/doc.go` (the diff keeps `ResolveParent` for "a reader that needs the pair itself").
   - One caveat: D13 names its readers explicitly (batten's notices, `noticesReachDriversParent`, loom's `primeOrchRecorded`, `NotifyPrime` callers) and doesn't list these. I read "parent names" in the decision's Scope line as covering them, so this is MEDIUM rather than higher.

2. **NIT, class `consistency`:** the word "umbrella" appears in added lines.
   - It is the project slug `umbrella` in the rename-test fixture and its comments and assertions (`internal/boardengine/rename_test.go`, added lines `newRenameBoard`). The record says beam throughout, never umbrella, in every name, text and doc the task touches.
   - It is a test-local fixture name, not a product name, so I rate it a NIT.
   - Other hits in the added lines are the unrelated board slugs `umbrella-daemon` and `umbrella-anchor`, which the record's Scope names.

**Tool calls (4 of 6)**
1. Grep of non-test callers of `RefuseUnlessPrime`, `NotifyPrime` and `ResolveParent`, plus added "umbrella" lines. My context held only the diffs I had read.
2. Read `loomcli/wiring.go`, `loomcli/decision.go` and `battencli/land.go` around the remaining `ResolveParent` and `NearestLiveOrch` sites, to see whether they are parent-pair or notice readers.
3. Read `landingdeps.go` and the `drivewiring.go` and `wiring.go` diffs, plus a grep of `primeOrchRecorded` and `noticesReachDriversParent`, to confirm the notice swap and look for other parent-name readers.
4. Grep of `ParentName` in hubgeom, orchcli, battencli and shuttleengine, plus a diff of `hubgeom/doc.go`, to confirm `ParentName` still comes from `ResolveParent` and the stencil/docs wording.
