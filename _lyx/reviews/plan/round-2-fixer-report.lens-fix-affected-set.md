```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran each finding's search to check that the fix covers every member of the set the finding is about.

Findings checked:
- **F3** (sites that remove the move record): covered. The three cards 15/18 sites now go through `RemoveMoveRecord`, and no other plan site removes the record.
- **F2** (destructive primitives prescribed outside `destroy.go`): covered. Card 14 was the only production site. The other hits are by-hand way-forward text (card 14 line 27) and test setup (`git rm` in cards 08 and 18).
- **F4** (errors after the record is written that leave the record): partly covered; see findings 1 and 2.
- **F1** (refusals that name `lyx batten cancel` for an owned-elsewhere slug): covered. That covers card 17 and the overview. Card 11 line 37 talks only about entries stopped by `anchor_stopped`, so the owned-elsewhere case doesn't reach it.

Findings I left unchecked: none.

1. **MEDIUM** — card 17, the restart-by-record paragraph (lines 56–58, "An error after `BeginAnchorChange` wrote the record decides the restart by the record…").
   The list of errors after the record is written leaves out two steps that also run after it: reopening the log sink after `Apply` returns successfully, and a `Finish` error.
   `Finish` itself does `RemoveMoveRecord` and `SpawnPairPush`, and either can fail.
   The card gives no end state for either one: no dobby restart decision and no re-run named.
   Evidence: card 17 line 55, "calls `Apply`, and reopens the sink at the anchor `Apply` left", and line 59, "Then `Finish`, then it starts the worktree's dobby…".
   The only failure handled after `Finish` is a failed dobby start (line 60).
2. **LOW** — card 18 Intent, lines 39–40, `Topology.add` with `Anchor`.
   This is the second writer of the record, and it is a member of F4's set "errors after the record is written".
   It writes the record, then runs `moveWeftContent`, wiring, scratch link, launchers and the origin record, and only then calls `RemoveMoveRecord`.
   No end state is stated for a failure in between.
   It is recoverable, because the re-run of `lyx beam create <slug>` goes through `EnsureAnchor`'s standing-record branch (line 45), but the card never says the Add error leaves the record and names that re-run.

Outside my lens: card 14 still lists `internal/fabricengine#cherryPickOnto` under Uses, but its Intent no longer calls it.

Tool calls:
- Grep for record-removal phrasing in cards 00–19: rerunning F3's search, since my context held only the sites I edited.
- Grep for prescribed destructive primitives across the cards (twice, because the first `-o` pattern dropped matches near the start of a line): rerunning F2's search over cards I had not read.
- Grep of card 17 for `Finish`, reopen and `BeginAnchorChange`, and of card 18 for failure text: rerunning F4's search for steps after the record is written.
- Grep for `batten cancel` across the cards: rerunning F1's search over the cards I had not read.
