```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix's change set for agreement with its surroundings.

1. **LOW** — `internal/hubreconcile/walk.go`, `carryMovedRunSettings` failure Warn. The carry now also renames retired `beam.pull_request` values, but its Warn still describes key moves only.
   - Evidence: the Warn says the `run_settings` "were not carried to their new key names", with way_forward "the next verb that loads the loom or tapestry config retries the carry".
   - A failed carry can now leave an `on` or `merge` value that refuses the entry's seed, and neither the message nor the way forward mentions it. The function's doc comment, `hubreconcile/doc.go` and the success mail were all updated.

2. **LOW** — `docs/overview.md:1391`. The push on re-entry changed in `prreview.go`, but its twin in the overview was not updated.
   - Evidence: the overview still says "The row pushes the task branch and opens or finds the pull request", with no exception.
   - The `PRReview` type doc gained "A re-entry whose record holds another gated head than the task head leaves that push to the wait, behind its gate." Line 1397 of the overview was edited for the gate halt only.

3. **LOW** — `pattern/PATTERN-config-layers.md`, "Config reinterpretation". The board `run_settings` rename (F3) runs on every walk, behind no board marker, and the pattern has no wording that covers a value rewrite there.
   - Evidence: the rule says a meaning change is applied by "only `hubreconcile`'s walk ... once per hub behind its board marker". For `run_settings` it names only key moves: "which ... `hubreconcile`'s walk carries to the board's `run_settings`".
   - `hubreconcile/doc.go` now says "It renames a renamed beam pull_request value there too, on every walk, since no layer admits one."

Tool calls (4 of 6):
- `git grep` with a `:!_lym` pathspec: it failed on the pathspec, so nothing was read.
- `git grep` for twins of the changed phrases: my context did not hold the repo-wide matches.
- `sed -n` of `docs/overview.md` 1386-1396, in the same call as the next item: my context held only some of those lines.
- `PATTERN-config-layers` rule text and a grep of `run_settings` in the docs and patterns: the rule line was in context, and the other matches were not.
