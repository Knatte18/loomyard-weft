```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for agreement with its surroundings and for twins left unchanged (lens fix-consistency).

**Findings**

1. **LOW** — `internal/boardengine/render.go:128`, the README intro line that every board file renders. It still says "An entry waits only on the open entries it names under After".
   - F4 rewrote the matching sentence in the `render.go` file header to "the entries each waits on that are not done, a project judged by its landing members". This twin was left as it was.
   - After the F1 and F4 rules, a dependency on a plain project whose members are all done is dropped from After. A dependency on an open project is shown by the project's handle, and a project is not an "open entry it names" in the sense the line reads.
   - The new seventh row of `TestRenderReadmeGolden` in `render_test.go` pins this line verbatim, as do the other rows. A fix would have to change the golden text in every row.
   - The review rejected a similar finding about the `doc.go` Ready paragraph, but not this rendered line, so it stays a LOW finding rather than a duplicate.

2. **LOW** — `internal/boardengine/render.go:137`, `readySection` meaning line "Waits on nothing open; can start now."
   - This is the same stale-twin class as finding 1. The wording is generic and arguably still true, and the review accepted a similar case for `doc.go`.
   - I report it only so the parent weighs it together with finding 1.

**Checked, no finding**
- `dependency.go`: the new doc comments on `met`, `metBelow` and `standsForBelow` agree with the `Dependencies` type comment and with the `layer.go` doc comment ("cycle detection follows a project to its landing members").
- `layer.go` is unchanged. `getDepth` follows the same unmet edges as `detectCycleDFS`, which now terminates on a loop and returns a cycle error before `getDepth` runs. The new `TestComputeLayers` rows assert that error, so the two agree.
- The `render.go` header edit agrees with the `afterCell` doc comment.
- The new rows in `dependency_test.go`, `siblings_test.go` and `render_test.go` agree with their test-function doc comments, which were updated where the fixer added a row.

**Tool calls**
- `grep -rn "open entr\|waits on nothing open\|Waits on nothing open"` over non-test `boardengine` sources: my context held `render.go` only at lines 1–40 and 570–620, so it lacked the section-text constants.
