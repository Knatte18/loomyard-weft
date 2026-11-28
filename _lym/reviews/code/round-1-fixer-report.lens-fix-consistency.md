```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of the fix change set against its siblings and twins.

1. **MEDIUM.** The F1 sweep rewrote `battencli/wire.go` to `gate.fixer_timeout_min` but left its twin bare.
   - `internal/gatecli/cardfixer.go:143` still reads "its timeout is the gate config's fixer timeout".
   - This is the `the <module> <key>` phrasing F1 names, and the exact words the fix replaced in `wire.go:802`.

2. **MEDIUM.** Comments naming `shuttle.engine` by phrasing were left, although F1 lists `engine` among the one-word keys to search after a possessive.
   - `internal/orchcli/remotecontrol.go:33` reads "whatever shuttle.yaml's default engine is".
   - `internal/shedrecipe/recipe.go:146` reads "DefaultEngine names shuttle's default engine".
   - `internal/shuttleengine/requires.go:47` (seen in the parent's earlier grep) reads "so the engine came from shuttle.yaml's default".
   - `_lym/landing/summary.md` says one-word keys such as `shuttle.engine` are named in full.

3. **LOW.** `internal/orchcli/remotecontrol.go:134` reads "whatever orch.yaml's switch says" on the same line where the sweep's neighbouring name is `orch.remote_control_name`.
   - The switch is the key `orch.remote_control`, which the sweep wrote in full in `orchengine/revival_test.go` and `reinterpret_test.go`.

4. **LOW.** `_lym/landing/summary.md` now says the guard skips "a name followed by a colon and at most one word".
   - `isDocumentField` does not skip a colon followed by a space and nothing else: `value == " "` is not empty and has zero fields, so the name is reported.
   - The `TextJoinedAfterColon` row pins that, so the summary's "at most one word" (zero included) is too wide.

Tool calls:
- One `grep` of comments for leftover possessive and `<module> <key>` phrasings, plus `git status --short`: my context held only the pre-sweep candidate lists, not the comments' current phrasings.
