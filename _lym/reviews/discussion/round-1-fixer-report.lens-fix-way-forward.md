```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward: checking that every refusal, error, next step, row and test the fix added can be carried out from the state it names and leads to progress.

**F-a: MEDIUM.** The new D6 acceptance criterion cannot pass for beam files as worded.
- **Changed text:** "D6: a regeneration with no workspace-content hook installed writes workspace files holding only `folders` and `settings`."
- **Problem:** No regeneration writes a beam workspace file holding only those two keys.
- **Evidence:** `internal/fabricengine/hubworkspace.go:227-230`. `markBeamWorkspace` always writes `"lym.generated": true` as the first top-level key of every beam file. Scope the criterion to "no `tasks` key, or no member beyond those the regeneration writes today", or name the mark.

**F-b: MEDIUM.** D5's refusal way forward fails from the subdirectory case the fix made explicit.
- **Changed text:** D5 bullet 1, "resolves the `open` verb's cwd with `lymcwd.ResolveWithin` instead of the `lymcwd.Resolve` it runs for the other board verbs … so the verb runs from anywhere in a worktree". This combines with the unchanged bullet 2 refusal naming `lym board rerender`.
- **Problem:** `lym board open` run from a subdirectory with no `board.html` names `lym board rerender`. That verb keeps the strict `lymcwd.Resolve` gate, so it fails from the same directory.
- **Evidence:** `internal/lymcwd/lymcwd.go:77,84`. `Resolve` "requires cwd to equal the anchored directory exactly" and returns `ErrCwdOutsideAnchor`.
- **Way forward:** The refusal should name the anchored directory to run `lym board rerender` from. The verb holds that directory in its resolved layout. The VS Code task runs at the folder, which is the anchor, so only the terminal path is affected.

**F-c: LOW.** The D3 downgrade outcome the fix introduced has no next step.
- **Changed text:** D3 Bound, "that commit replays as a plain cherry-pick rather than through the by-entry merge, and can conflict."
- **Problem:** The old text said the case heals. The fix replaced that with a conflict that names nothing for the operator to do.
- **Evidence:** `internal/fabricengine/replay.go:702-703`. A cherry-pick conflict returns `ErrReplayConflict` with the starting tip restored. The bound should say what a conflicted replay leaves and how it is resolved, or point to the existing way forward `ErrReplayConflict` carries.

No other way forward the fix introduced fails:
- The D6 `"reveal": "silent"` change surfaces the existing refusal.
- The D6 Bound's "next regeneration or seed by an up-to-date binary restores them" is reachable.
- The hook is installed in every non-test process (`cmd/lym/main.go:158`), so production regenerations always run it.

Tool calls:
- Read `internal/lymcwd/lymcwd.go:70-164`, for `Resolve`'s and `ResolveWithin`'s gate behaviour, which my context lacked.
- Read `cmd/lym/main.go:135-161`, for the condition guarding the installs near the namer, which my context showed only from line 160 on.
