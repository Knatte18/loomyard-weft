```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. I checked what each card's change forces outside the members it lists.

- **LOW, Card 7 (Decisions 1 and 2), `Extends` on a ref the card rewrites.**
  Card 7 lists `internal/dobbyengine/doc.go` under `**Extends:**`, but its change makes existing loop bullets false, so they must be rewritten:
  - `internal/dobbyengine/doc.go:309` says the loop "waits lock.ProbeGrace … and returns ErrDobbyRunning".
  - `internal/dobbyengine/doc.go:311` says "A state record holding the process's own pid is a re-exec … any other pid, or no record, is a start".
  Both are replaced by the start-ticks test and the ten-second re-exec wait.
  Fix: move `internal/dobbyengine/doc.go` under `**Edit:**` on card 7.
- **LOW, Card 3 (Decision 8), stale docs of the landing count.**
  After card 3, landings are counted for this machine only.
  Two texts still say every Land Done counts, and no card lists them:
  - The `Deps.Landings` field doc, `internal/dobbyengine/loop.go:90`: "the slugs of the runs whose parent worktree is this dobby's own and whose batten history holds a Land Done, whatever the run's state".
  - The board-tidy bullet, `internal/dobbyengine/doc.go:371`: "records each Land Done as counted".
  Card 3 lists only `internal/beamcli/doc.go`.
  Fix: card 3 also lists `internal/dobbyengine#Deps` and `internal/dobbyengine/doc.go`.
- **LOW, Card 9 (Decision 6), dead method and stale seam doc once the `pass` line goes.**
  - `internal/dobbyengine#passKind.String` has one caller, `internal/dobbyengine/passes.go:251`, which is the `pass <slug> <kind>` line card 9 removes. Its doc, `internal/dobbyengine/passes.go:67`, says "String names the kind as dobby's log line does". After card 9 the method is dead and its doc is false, and no card lists it.
  - The `Deps.Log` field doc, `internal/dobbyengine/loop.go:104`, says "one for each task start and each pass spawned". Card 6 edits `Deps` but its Intent does not say it drops that text, and card 9, which changes the lines, does not list `internal/dobbyengine#Deps`.
  Fix: card 9 lists `internal/dobbyengine#passKind.String` (as a delete, or reworded if it is kept) and `internal/dobbyengine#Deps`.

Calls made beyond my context:
1. grep for the users of the `CheckBinary` and `Log` seams and the callers of `hubDobbyDeps`, `DobbyLandings`, `startSuite` and `LogPath`, and for `TestDobbyGeometry`: my context had not searched any caller sets.
2. grep for `TiersValue`'s finalize assertion, the `twoTier` fixture, the test files that use gate keys, `beamcli`'s importers, `x/sys` in `go.mod`, import allowlists, and `gatecli`'s imports: to check the forced edits of card 11 and cards 13–15 and import cycles, which my context had not checked.
3. The package clauses of the test files the Create handles go into, and the `finalize` uses in hubreconcile, configcli, hubforge and plugins: to check handle packages and fixtures, which my context lacked.
4. The adopt and dobby mentions in the board skill, the plugin skills list, and the command-tree tests: to check skill and CLI-index forced edits, which my context lacked.
5. The `clitree_test` hidden-command filter and the index and annotation tests: to confirm a hidden verb needs no index entry, which my context lacked.
6. `internal/loomcli/wiring_seams_test.go`'s seam table, the adopt integration test's suite assertions, the callers of `passKind.String`, and the beam status test's envelope compare: to check the forced edits of cards 1, 9, 14 and 15, which my context lacked.
