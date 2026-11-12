```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every refusal, next step, row or test the fix introduced can be reached and leads to progress.

No findings.

What I checked:
- **D11, first prime call:** the operator runs `lym gate testmap` by hand in the prime. That works on the owner machine. On a non-owner machine the call is refused, and the refusal names the owner machine as the way forward.
- **D11, removal on a non-owner machine:** it can happen once that machine's pulled weft holds a committed map. A non-owner never writes maps, so this waits on the owner's commit and the weft pull, and both are things that already happen.
- **D11, rows for a killed import and for a non-owner call:** both can be reached from the states they name. The next call skips packages that already have a map, commits the leftover maps per D4, and only then removes old maps.
- **D9, a task's pass under the task's slug:** a call can get its slug, because the slug comes from `Location.WorktreeName` (`internal/hubgeom/hubgeom.go:108`). The teardown ends processes through `runproc.EndSparing(..., slug, ...)` (`internal/pairteardown/teardown.go:698`), so the new D9 row (a task's pass is ended by its pair's teardown) can be carried out.
- **D7, audit row:** the planned HEAD is the pass's own `r.head`, which the exit record stores as `planned_head`. The audit can read it inside the pass.

Further tool calls:
- grep of `internal/lymcwd/*.go` for a slug on `Location`: my context did not show whether a gate call knows its task's slug.
- read of the `Location` type and grep for slug helpers in `internal/hubgeom`: the first grep showed no slug field.
- grep for `WorktreeName` in `runproc`, `pairteardown` and `hubgeom`: to confirm the slug comes from the worktree name.
