# Card 8 — Sweep the docs, the sandbox suites and the #673 workaround

**Prosa:**
- `internal/reedengine/doc.go`
- `internal/reedengine/render/types.go`
- `internal/reedcli/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-REED-SUITE.md`
- `tools/sandbox/SANDBOX-REED-WATCH-SUITE.md`
- `CLAUDE.md`
- `internal/reedengine/watchdog_test.go`

**Intent:** D8: every doc loses its claim that a strand is split under a parent or stacked in the shell window, and the #673 workaround goes.

`internal/reedengine/doc.go` is rewritten where it describes the stack, its height rules and pins, the `window-resized` array and the resize self-heal, the attach chain, the zoom bracket, Alt+z, the strand pane buttons and the pane-border title, and the measurement record of repaint candidates that measured the pins' artifact.
It states the rules as they now stand: every placed strand gets an own window, a group window or the overview; the shell window holds Selvage alone, which fills it after boot, a heal and the move; the reconcile head moves a strand pane left in the shell window into its own window (card 7's rule, the last-pane bound included); boot and attach clear the `window-resized` hook; the per-hub daemon runs name repair, gated by `watchdog`; the bindings unbind `M-z`; a retired anchor in `reed.json` reads as `own-window` and is refused on write.
render's package doc, in `types.go`, describes the package as the closed display vocabulary (`Anchor`, `Display`, `ValidateDisplay`) and drops the `Rules` contract, the policy and mechanics layers and the five-anchor list.
`internal/reedcli/doc.go` drops the stacked-layout and resize-daemon wording and names `--anchor own-window|hidden`.
`docs/overview.md`'s reed row says reed hosts every managed process as a strand in a window of its own or its group's, with no "arranges them" or own-window-as-the-exception wording, and calls the watchdog the per-hub name-repair and reap daemon; its spawn walk-through's step 4 no longer names `anchor:below-parent`.
Its `lyx ide spawn` bullet says the VS Code block forwards the four Alt keys of reed's bindings, and its "reed is three things" bullet drops the positional stack heights, `collapsed_rows` and the content and sharing sizes, saying a display is an anchor, focus and an optional window group, and that render is the closed display vocabulary rather than `layout = Rules(strands, box)`.
The `internal/reedengine/watchdog_test.go` header stops listing `resizeHookCommand`, `tmuxQuoteValue` and `resizeSignalPath`, whose tests card 2 deleted.

Both reed sandbox suites are swept: a scenario that exercises only the retired layout or the resize self-heal (M12's stack geometry, M18's mother/child shrink, M26's resize self-heal, and the watch suite's stack-collapse and live-resize scenarios) is retired with a one-line reason as M6 already is, and a scenario that adds a strand for another purpose adds it with the default anchor and watches its own window (PATTERN-sandbox-coverage keeps reed exercised).
`CLAUDE.md`'s #673 workaround bullet goes, and the `## Workarounds` section with it, since nothing else is in it.

After this card, outside the CLI refusal, the engine refusal and read-time remap, the retired-key fixtures, the sandbox retirement notes and `docs/benchmarks/`, no `.go`, `.md` or `.yaml` file names `below-parent`, `AnchorBelowParent`, `AnchorTop`, `--anchor top` or `anchor: top`; the plan's verify section greps for the two constants, for `#673` in `CLAUDE.md`, and for the three other spellings in every `.go`, `.md` and `.yaml` file outside the exception files it names, so a leftover in any other file fails verify.

**Commit:** 8: docs: sweep reed's pane-era placement out of the docs and sandbox suites
