```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F3, F4, F5, F6), looking for concrete cases where a proposal is wrong.

no findings

- F3 (grouped placed-apart strand follows the Title rule): this changes no window name today. The only `render.Display` built with `Title` is the runs strand's overview spec (`internal/orchcli/runs.go:44`). The break-out path keeps the strand's display record, so it falls under the same rule, and a shared window keeps its group name as before.
- F4 (criterion condition matches D4): a halted or paused run draws the line with its state, which D4 names. No sibling depends on "live".
- F5 (collision facts): the only lookup by window name is `OpenWindow`'s `windowListFormat` (`internal/reedengine/window.go:41`), which skips windows with `@lym_own`. The status-bar formats only render `window_name`. The shell window is found by its console pane. No case turns a shared name into more than two buttons with the same name.
- F6 (drop the `PATTERN-no-hard-coded-choice` label): the rule now stands as the record's own. The support-log rejected alternative was relabelled with it, and `PATTERN-loom-contract` still justifies that rejection.

Tool calls:
- grep for `Title:` composite literals and window-name lookups under `internal`: my context did not show which callers set `Display.Title` or which code reads `window_name`.
- grep for `.Title =` and `"title"` in reedcli, reedengine and loomcli: to rule out a CLI flag that sets a non-overview `Display.Title`, which my context did not show.
