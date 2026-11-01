```yaml
lens: fix-proposal
outcome: reported
```

I checked each reviewer proposal the fixer carried out (F1–F7) and tried to build a case where the proposal itself is wrong.

**Findings**

1. **MEDIUM. F2: the render reads `heddle.json` under the Bolt write lock.** Changed text: D3, the bullet starting "It reads `heddle.json` holding the Bolt write lock … taken only around the read".
   - Summary: a foreground render can be stuck behind the network work of a background Bolt push. The decision states no wait limit and no outcome for a lock it cannot get.
   - Evidence: `retryBoltPush` (`internal/fabricengine/pushclassified.go:677-695`) holds the board write lock across a `repo.Fetch()`, the replay and the push. `PushRecorded`'s doc says that retry "runs again over the push backoff". The board's detached sync takes this path after every heddle or board write.
   - Concrete case: a `lym fabric pull` that takes in an `AGENTS.md` change fires the take-in render. The render then blocks on `lock.AcquireWriteLock` (it blocks with no time limit) until the background push's fetch and backoff finish. The fabric verb hangs, even though D6 says a seam failure never holds up the verb.
   - What the decision should add: a bounded wait (`lock.AcquireWriteLockWithin`). Running out of time is a seam failure, which D6 already reports as a warning naming `lym heddle render`. Add a matching acceptance case.

2. **LOW. F3: log-only for `ReconcileOrigin`.** Changed text: D6, "Where the move's caller has no result to carry the warning, as a code-branch fast-forward in `ReconcileOrigin`, whose `OriginReport` holds no warning, called from `battencli` or `hibernatecli`, the failure is log-only".
   - Summary: the proposal treats those callers as having no foreground report. They do have one, so a foreground verb's warning disappears, against D6's own "a foreground verb also reports it in its result".
   - Evidence: `internal/orchcli/start.go:346` prints `report.Summary()` from a foreground `ReconcileOrigin` call. `hibernatecli/reconcile.go:46` is the foreground `lym hibernate` verb. `OriginReport` already reaches the operator through `Summary()` (`internal/fabricengine/originreconcile.go:246`).
   - What the decision should do instead: give `BranchOutcome` or `OriginReport` a warning that `Summary` prints. Log-only should be kept for a process with no report at all.
   - Naming only `battencli` and `hibernatecli` also leaves out the other callers: `orchcli`, `adoptcli`, `loomcli` landing and `fabriccli` remove. Under the new wording they are log-only too.

3. **LOW. F7: the refusal's way forward is `lym fabric list`.** Changed text: D2, "a name that is none of the hub's warp worktrees, as `lym fabric list` lists them, is refused", and the refusals line naming `lym fabric list`.
   - Summary: the verb takes a worktree name, but the way forward lists paths.
   - Evidence: `lymcwd.NamedWorktree(hubPath, name)` takes a name. The `list` command's Long text in `internal/fabriccli/fabric.go:174-178` says it "outputs warp worktree paths only".
   - What the decision should do: state that the name is a listed path's last element, or accept a listed path as well. Otherwise an operator who pastes a line from `lym fabric list` is refused.

4. **LOW. F1: the `.heddle` temporary directory.** Changed text: D3, "removes the emptied directory … the render leaves no file beside its targets"; D5's template default `.heddle`.
   - Summary: there is no rule for a repository that already has a `.heddle` path at its root.
   - Evidence: D3 guards a target file without the marker, but never guards the temporary directory. An exclude line does not hide a tracked path. If `.heddle` is a tracked directory, the directory is never emptied, and a temp file left by a crash sits in a tracked tree. If `.heddle` is a tracked file, the render cannot create the directory, so every render fails. The decision names no refusal or warning for either.
   - What the decision should do: name the `.heddle` collision, or choose a temp name tied to the target, under the same marker or skip rule.

Proposals with no case found: F4 (the `.claude/AGENTS.md` import), F5 (the leaf-packages background edit) and F6 (the wider test exception).

Tool calls:
- I read `internal/fabricengine/pushclassified.go:665-700` and grepped the `ReconcileOrigin` and `Summary` call sites in `hibernatecli` and `orchcli`. My context had only the `retryBoltPush` signature line, not how long it holds the lock, and did not show whether the reconcile callers print a report.
