```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. Here are the forced edits no card lists, and the sets the cards cover only in part.

1. **MEDIUM, cards 3 and 4 (D3/D4): `docs/overview.md` still states the old placement, and no card lists the file.**
   - `docs/overview.md:544` and `docs/overview.md:821` say `lym loom start` puts the status strand "as the session's overview window, or in a window of its own that never takes the viewer when a beam's `runs` strand holds the overview". Card 4 removes that branch.
   - `docs/overview.md:687` says ensuring the runs strand "replaces a `loom-status` strand a pawl run's `lym loom start` put in the overview window first". Card 3 removes that replacement.
   - Fix: card 3 lists `docs/overview.md` under `**Edit:**` for line 687, and card 4 does the same for lines 544 and 821.

2. **LOW, card 1 (D1, and the Scope line "every package doc and comment in reed … that states one overview strand per session"): reed comments still describe a single overview window that no card lists.**
   - `internal/reedengine/windoworder.go:1`: the file header reads "the overview window, agent windows in start order, …". Card 1 names only members of this file, not the file itself.
   - `internal/reedengine/bar.go:4-5`: "the overview window among them" and "Line 1 follows window index order: the overview, the agents …".
   - `internal/reedengine/doc.go:109` ("the overview's button is first") and `internal/reedengine/doc.go:122` ("the session's first window, the overview"). Card 1 rewrites only the "Overview anchor" and "Window order" paragraphs of `doc.go`, and card 2 only the follow and focus paragraphs.
   - Fix: list `internal/reedengine/windoworder.go` and `internal/reedengine/bar.go` (file self glyphs) on card 1. Name the two extra `doc.go` sentences in the Intent: overview windows, in ascending order key, come first.

3. **LOW, card 2 (D2): the doc comments of the focus rule's callers still say "the overview" in the singular.**
   - `internal/reedengine#Engine.FollowRun` (`follow.go:30`): "else the overview, else nothing".
   - `internal/reedengine#Engine.selectRelevantWindowIfCurrentLocked` (`ownwindow.go:514`): "lands on the newest live agent or the overview".
   - Card 2 rewords only `doc.go`'s copy of the `FollowRun` sentence, not these comments.
   - Fix: add both member glyphs to card 2 and reword them to "the lowest-index overview window".

4. **LOW, card 4 (D3): `internal/orchengine/doc.go:14` says the runs strand is "the session's overview window" and that "the beam's window is the first window".**
   - The second half stays true, because the runs key is zero. The first half now reads as the only overview window.
   - Fix: reword it to "the session's first overview window". Card 3 is the natural carrier, but no card lists the file.

5. **LOW, card 1 (D1): `TestUpdateStrandLocked`'s doc comment does not list the order refusal.**
   - The comment (`internal/reedengine/strand_test.go:253-257`) lists every refusal the test pins, and card 1 adds the `OverviewOrderChangeRejected` row without extending that list.
   - Fix: card 1's Intent extends the comment with the order refusal.

Calls made:
- A grep for stale overview and loom-status mentions across Go and Markdown, and for the helper glyphs card 4 lists under Uses. My context had no repository-wide search beyond reed, orchcli and loomcli.
- A narrower grep listing the files that mention an overview window, strand or anchor, and where `seedLLMDriver`, `tmuxBinaryPath` and `windowOfPane` are defined. The first output was truncated.
- Overview mentions in `docs/overview.md`, `bar.go`, `bar_integration_test.go`, `ownwindow.go`, `spawn.go` and `windoworder.go`. None of these spans were in my context.
- Overview mentions in `orchengine/doc.go`, `loomcli/bootstrap.go`, `follow.go` and `spawn.go`, plus any skills. `orchengine/doc.go` had not been read.
