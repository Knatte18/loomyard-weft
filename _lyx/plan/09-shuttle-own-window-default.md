# Card 9 — a shuttle run's strand opens in its own window by default; the orch stays in window 0

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#Spec.validate`
- `internal/shuttleengine#normalizeAttachSpec`
- `internal/shuttlecli#shuttleCLI.runCmd`
- `internal/orchcli#orchCLI.orchSpec`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestSpec_Validate`
- `internal/shuttleengine#TestAttach_StrandLaterLosesPaneBinding`
- `internal/shuttleengine#TestAttach_OutcomeDisposition`
- `internal/orchcli#TestStart_NoStrandLaunchesAndSpawnsWatcher`

**Uses:**
- `internal/reedengine/render#AnchorOwnWindow`
- `internal/reedengine/render#AnchorBelowParent`

**Intent:**
D10: every agent a driver spawns gets a window of its own, by one default in shuttle, while the sessions that live for a whole run or the whole hub name their anchor and stay in window 0.

- `Spec.validate` defaults an empty `Display.Anchor` to `render.AnchorOwnWindow`; its doc comment and the `Spec.Display` field's doc comment say so, and that a long-lived session names `below-parent` explicitly.
- `normalizeAttachSpec` defaults an empty anchor to `own-window` too, so run and attach agree.
  Attach never relaunches or moves a strand: the anchor only feeds the reconstructed run's liveness classification, where `own-window` and `below-parent` are both non-hidden and so classify a cleared binding the same way, so attaching to a strand an older binary recorded in window 0 leaves it where it is.
  The comment above the default says so.
- `shuttleCLI.runCmd`'s `--anchor` flag defaults to `own-window`, its help reads `placement: own-window|below-parent|hidden`, and its `Long` says a run's strand opens in a detached window of its own unless `--anchor` says otherwise.
- `orchCLI.orchSpec` sets `Display{Anchor: render.AnchorBelowParent, Focus: true}`, so the orch keeps its window 0 pane.
- The loom driver (`internal/loomcli`'s `driverSpec`) already names `below-parent` and needs no change.
  Every other `shuttleengine.Spec` site in the tree spawns a producer agent that ends with its row (the discussion, plan, rework, describe and darn writers, webster's master and recovery agents, the bouncer, treadle, alignment, friction, merge-resolve and the single-LLM producers), and so takes the new default; the seat engine names its own window group (card 10).
- `internal/shuttleengine/doc.go` says a run's strand lands in its own window unless its spec names an anchor, and names the orch and the loom driver as the window 0 sessions.

Tests:

- `TestSpec_Validate`'s row `anchor defaults to below parent` becomes `anchor defaults to own-window`, asserting `render.AnchorOwnWindow`; the pass-through row keeps `below-parent`.
- `TestAttach_StrandLaterLosesPaneBinding`'s doc comment and `//testtiming:keep` reason say an empty anchor is defaulted to `own-window`; its rows are unchanged, since the defaulted anchor still surfaces `errStrandPaneBindingCleared`.
- `TestAttach_OutcomeDisposition` also asserts no `AddStrand` call on every row, so the `running_attaches` row pins that attaching under the `own-window` default to a live strand reed reports without the own-window flag (a window 0 strand) neither moves nor refuses it.
- `TestStart_NoStrandLaunchesAndSpawnsWatcher` asserts the orch spec's `Display.Anchor` is `render.AnchorBelowParent`.

**ImpactSummary:** One default flips in `Spec.validate`, `normalizeAttachSpec` and the `lyx shuttle run` flag; every spec site without an anchor now spawns into its own window, and the orch gains the explicit anchor that keeps it in window 0.

**Commit:** 9: Shuttle runs open in their own window by default; the orch names window 0
