```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F2, F3, F5, F6, F7) by trying to build a concrete case where each one is wrong.

no findings

I tried to break each proposal and none of the cases held:

- **F1:** `battencli`'s `PrepareRunSettings` path is the `deps.PrepareRunSettings` that Seed-Child wraps. So the Seed-Child refusal still prints one way forward. `lym board upsert`'s `validateRunSettings` returns the refusal on the write itself, which fits "repeat with the new name".
- **F2 (move file and base before the follow):** I checked four cases against FollowTemplate's rules as quoted in `follow.go`, and all hold:
  - An untouched block: moved file equals moved base, so it takes the template.
  - A shipped value the operator edited to 3: it differs from the moved base, so it is kept.
  - No base: the moved file's value is kept and listed under Differs.
  - A block the operator added: it is not in the template, so it is kept.
- **F3 (stale stamp on a D7 failure):** a persistent D7 failure reruns the walk at every start verb. That is the same cost a skipped hub-wide write already has. A malformed single entry does not fail the read, because records are read tolerantly (`PATTERN-tolerant-records`), so I found no permanent-failure case unique to D7.
- **F5:** a prime copy holding both an old key and its new path falls under D5's new-path-wins rule. The prime's own worktree file is still moved later by `reconcileWorktree`, so no value is lost.
- **F6:** tapestry's `plan` and `parts` were bare scalars (D1's table), so adding them to the refusal list is consistent.
- **F7:** YAML value equality ignores mapping order and comments, so after the follow an untouched file matches the new template.

Tool calls:
- Read `internal/battencli/runsettings.go:30-60`, `internal/boardcli/runsettings.go:135-160` and grepped `hubWriteGuard`: my context did not show how the other `PrepareRunSettings` callers wrap the refusal (F1).
- Read `internal/configcli/configcli.go:763-778` and grepped `ErrInvalidRunSettings`: my context did not show the hub write guard or which caller feeds Seed-Child's wrapper (F1).
