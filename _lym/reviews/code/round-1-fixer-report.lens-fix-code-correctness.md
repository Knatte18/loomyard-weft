```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code for correctness (lens fix-code-correctness).

I traced the changed functions on concrete inputs and found no BLOCKING or MEDIUM defect. The gate passing (0 failures) agrees with that. I did not run any mutants.

**Traces that held**
- `widen(bound)`:
  - Floor 82, doubled floor -86, bound `at(3)`: it clamps to 3, walks once more, then returns false because the floor is no longer after the bound. This is the F9 row's `[at(82), at(3)]`.
  - Zero bound with `walked == previous` still falls back to the whole history, the intended path for runs with no stop.
  - If the `lookback *= 2` or `b.walked` assignment is dropped, the zero-bound row walks with a zero `Since`, so the exact-list assertion catches both F6 mutants.
- `readBoardHistory`:
  - With an empty first-parent walk, `before = tip`, the tip is read once, and `history.before` is the tip's first parent.
  - With a walk that ends at the root, `history.before` stays empty, so `readOlderLabels` does nothing.
  - With a zero floor, `before` is empty too.
- `readOlderLabels`: the cursor advances by first parent, each older commit is read once, and it returns as soon as no slug is pending. The `page[len(page)-1]` index is safe because `Log` from a valid sha returns at least that commit.
- `checkpointFloor` and `taskCommitBelow`:
  - A zero `openedAt` gives a whole-history walk, which is what the fixture tags built with `fullRun` get.
  - The only unattempted tags are those in `pairOpened`, so every tag whose window the time join needs is covered.
- New rows:
  - Row 2: the floor lowers to 160 and `b1` at 170 closes the window, so `L` at 210 stays unmapped. Without the floor loop, `b3` at 215 would map it.
  - Row 5: without `readOlderLabels` the kind would be `other`, so it fails with the defect in place.
  - The walk-list rows 3 and 4 fail if the bound logic is removed.
- Callers: both `snapshotsOfRun` callers (`runs.go`, `timing.go`) pass `seed.StartedAt`, which is in scope in each. No other reader of `inputs.checkpoints` remains.

**Findings**

- LOW — `batten.go`, `snapshotsOfRun` `reach` from `startedAt`. For a run with no earlier tag, a batten seed committed more than `battenSeedMargin` before the loom seed's `started_at` is now never found, so the batten source is absent where the old full widening found it. This holds for landed and in-flight runs. The F3 fix proposed this bound, but it is an answer change that no test pins and the parent's F5 ruling ("never changes what the report answers") does not cover. Evidence: the `reach` computation in `snapshotsOfRun`, and the old `widen`, which ended only at the whole history.
- LOW — `batten.go` `walk` and `landings.go` `readCheckpoints`: both now use `FirstParent: true` on the records branch, so a checkpoint or status commit that reaches the main line only through a merge's second parent is missed. The code does not establish that the records line is linear here. The only support is that other records readers (`committedfile.go`) use first-parent. No test has a merge on the records line.

**Tool calls**
None. My context already held every changed file and the review.
