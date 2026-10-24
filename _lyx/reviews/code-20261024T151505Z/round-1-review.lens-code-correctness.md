```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `runs-never-write-board` diff.

**Per-function list (case run, verdict)**

- `isRunAgentSession`
  - `""`: Parse fails (one segment), false. Correct.
  - `ly:orch`: slug empty, false. Correct.
  - `ly:t:orch`: role is orch, false. Correct.
  - `ly:t:webster`: true. Correct.
  - `ly:t:orch-2` (a numbered second orch thread): `Role == "orch-2"` is not `RoleOrch`, so it is treated as a run agent. **Wrong.** See F1.
  - Malformed name such as `a:b:c:d`: Parse errors, false, so the guard is off. Matches the sibling guards, which also fail open.
- `checkTaskCreation`, `loaded`/`after` cases
  - New task: slug absent from `wasTask`, refused. Correct.
  - Note upserted to kind task: the loaded note is not in `wasTask`, refused. Correct.
  - Promote: refused. Correct.
  - Batch with a new task: the end state is checked, refused. Correct.
  - Merge upserting a new slug: refused. Correct.
  - Merge upserting a slug that was an existing task: passes (D1 consistent).
  - Edit of an existing task: passes. Correct.
  - Delete a task and re-create the same slug in one write: net unchanged, passes. Correct for a net-change check.
  - Duplicate slugs in `after`: not reachable.
  - Empty `loaded` and `after`: nil.
- `boardCriticalSection` placement
  - The call comes after `renameSlug(loaded, …)`, so a rename is judged under the new slug. Correct.
  - It comes after `checkRunLocks`, so a run-held-entry violation is reported first. Fine.
  - It comes before `store.Save()`. A refusal returns before the save and `fn` only mutated the in-memory store, so `board.json` is unchanged. The test pins this.
  - A `noWrite` result returns early, so no creation is possible there.
  - Replay of a local commit does not go through the critical section.
- `cli.go` `cfg.StrandName = strandNameFromEnv()`
  - It is set after `ApplySkipEnv` and before `boardengine.New`.
  - `board()` reuses the built `b`, so every write verb carries the name. Correct.
  - Other `boardengine.New(...)` constructions would carry no name; I did not check them, and the open-risk note covers this.
- `New` copies `cfg.StrandName` into `Board.strandName`. Correct.
- Refusal message
  - It contains `strandName`, `%q` of the slug, and `"kind":"note"`.
  - The `%s` of a strand name is fine. Correct.

**Findings**

- F1 (MEDIUM, design) `internal/boardengine/taskguard.go:14`: `isRunAgentSession` compares `name.Role != agentname.RoleOrch` exactly.
  - `agentname.NumberRole` produces numbered roles such as `orch-2`, and `agentname.MatchesRole(role, base)` exists for this case.
  - The sibling guard `internal/fabriccli/roleguard.go:45` uses `MatchesRole` for the orch role, which is the matching the decision record's D2 and its acceptance criteria point to.
  - A numbered second orch thread (`<shortname>:<slug>:orch-2`) is refused creating a task, contradicting "an orch thread writes as before".
  - Fix: use `!agentname.MatchesRole(name.Role, agentname.RoleOrch)`, and add a `ly:some-task:orch-2` row to `TestTaskCreationGuard` that passes.
  - I did not confirm `MatchesRole`'s exact semantics; I read only its call site.

No other defects found in the lens.

Tool calls (2):
1. `sed -n 56,135p internal/boardengine/board.go`: the manifest had only the diff hunks, and I needed the call order around `renameSlug`, `checkRunLocks` and `Save`.
2. `grep` of `sandboxedRoleRefusal` and the `agentname` role helpers: to see how the sibling guard matches the orch role, and whether `MatchesRole` and numbered roles exist.
