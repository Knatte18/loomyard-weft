```yaml
lens: fix-code-correctness
outcome: reported
```

Done entries put tasks before notes, so the done order is [soon task, later note]. With the old predicate P0 would land between them at index 2, and `lines[1]` would be `soon`, so the test fails with the defect in place.

**Lens fix-code-correctness: no findings.**

Cases I ran through step by step:
- **`IsDone`**: it reads `Layer == doneLayer`. `ComputeLayers` sets `doneLayer` for done entries. If it fails, the layer is empty and the entry counts as open, which is acceptable. `withRootProjectRow` is the only user, and it is called only from `list`, so no caller breaks.
- **`withRootProjectRow`**: with entries `[soon (done task), later (done note)]`, `IndexFunc` matches index 0 through `IsDone`, so P0 goes first. That gives 4 lines with `lines[1]` starting `P0 `. With no matching entry it appends at the end, as before.
- **`list` under `--sort`**: it appends P0 last, so the last line starts `P0 `. With the old `withRootProjectRow` call, the sorted list `[later note, soon task]` would put P0 first, so the sort assertion fails with the defect in place.
- **Label test**: with one filtered row the column widths collapse, and `[active]` follows after the column gap. That matches the expected string and the gate passed. If the `len(listLabels) == 0` guard were dropped, a P0 row would print and the exact-stdout comparison would fail.
- **Done-placement test**: this is the only test that depends on `RenderOrder`'s done ordering (the second case above).

Further tool calls:
1. Read `internal/boardengine/layer.go` lines 185–234. My context lacked `RenderOrder`'s done ordering (kind rank), which decides whether the done-placement test fails with the old predicate.
