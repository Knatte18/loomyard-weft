```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens, applied to the amendment's three merged files (`internal/driftcli/wiring.go`, `internal/loomcli/driftwiring.go`, `internal/loomcli/wiring.go`).

**Rules checked**

- `PATTERN-shed-recipe-registry`: held. Nothing in the three files touches the registry. The Drift-Await row and its `entry_gate` live in the recipe and `shedrecipe`, which are settled and out of scope.
- `PATTERN-refusal-way-forward`: held. The only refusal text left in `driftcli/wiring.go` is the "this worktree has no run status" refusal. It still names its way forward ("lym drift verbs run inside a loom or darn run's worktree"). The seed-missing refusal was removed together with the seed read, so no refusal lost its way forward.
- `PATTERN-sole-parsers` and `PATTERN-tolerant-records`: held. `verbStatus` still reads status only through `state.ReadJSON[shedengine.Status]`, which is the `internal/state` route. After the merge it has no `shedrun.ReadSeed` call and no recipe inspection.
- `PATTERN-cliwire-sole-wiring` and `PATTERN-cwd-resolution`: held. The wiring still takes paths from `w.location` and `location`, and the changed lines add no new path resolution.
- `CLAUDE.md` "Docs land in the same commit": held for these files. `driftcli/doc.go` and the other docs changed in the range, and the grep for `AwaitRow`, `WaitCapture` and `awaitRow` finds no reference left in the repo outside `_lym`. `ReadJSONNoMkdir` is also gone, so only `ReadJSON` remains.
- Comment convention on changed lines: held. The `verbStatus` doc comment now matches what the function reads (status, capture lock, agent-session mark). The `driftDeps` doc comment no longer mentions `awaitRow`.
- Commit messages and departures: held. The range has no `Pattern-Departure:` trailer and none is needed.
- Declared PATTERN changes: the directive declares none, so nothing was raised.
- Focus question 1 (`verbStatus` reads through the parent's `ReadJSON`, no `AwaitRow` or seed read): held. `go build ./internal/driftcli/ ./internal/loomcli/ ./internal/driftengine/ ./internal/shedrecipe/` and `go vet` on the first two printed nothing.
- Focus question 2 (parent's `ReadJSON` call sites sit beside the removed `awaitRow` parameter): held. `wiring.go:780` calls `driftDeps(location, statusPath, statusLockPath, driftTables{...})` with no leftover argument. The `ReadJSON` swaps at `driftwiring.go:61` and `wiring.go:933` are untouched by the task's lines.
- Focus question 3 (anything newer still using `AwaitRow`, `WaitCapture` or the `capture: wait` key): held. The repo-wide grep found no `AwaitRow`, `WaitCapture` or `awaitRow`. The pattern search for `capture: wait` found only unrelated `wait` hits (`--wait` flags, status-spec `activity.wait`, test names).

**Findings**

None.

**Outside my lens, not a finding:** the comment at `internal/loomcli/wiring.go:934` ("A run directory not yet created fails the lock open with not-exist") sits beside the parent's `ReadJSONNoMkdir` to `ReadJSON` swap. I did not verify whether `ReadJSON` still returns not-exist for an absent run directory. That line is the parent's change, not the task's, so I recorded no finding.

**Tool calls:** none. My inherited context already held the diffs, the grep results and the build output.
