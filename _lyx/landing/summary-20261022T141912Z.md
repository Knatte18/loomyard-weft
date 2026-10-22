# Report's exact join stops at the run's pair-opening commit

`lyx report convergence` maps an archive tag to its landing first through the exact join, which reads the task commit from the newest task-commit trailer in the first-parent records history below the tag.
A darn run writes no such trailer.
Its walk went past the run's own commits into the parent's records history, where the parent's sync commits carry trailers naming the parent's own landings, and took the newest of them.
Every darn run tagged since that sync mapped to the same older landing, de6b8465f, so the report filed a "claim one landing commit" finding and left all of them unmapped.

The walk now ends at the commit that opened the run's pair, the origin-record commit `fabric: record parent branch for <slug>`.
A run whose own history holds no trailer finds no task commit, is not exact-attempted, and maps through the time join: its window opens at its landing checkpoint and closes at the board commit marking its entry done.
A run whose trailer sits above that commit, as a loom run's Webster trailers do, reads the same task commit as before.

Closes #831

## Design decisions

- **The floor is the pair-opening commit, matched by slug.**
  A pair's records branch forks from its parent's, so everything below that commit is the parent's history.
  A history with no such commit, from a pair older than the commit, walks as before.
- **`fabricengine` exports `OriginRecordSubject` and `OriginRecordSlug`**, and pair creation commits the origin record under `OriginRecordSubject`, so the report reads the subject through a neutral export, as it already reads the archive tag and the trailer.
- **A darn run maps through the time join, not the exact join.**
  No darn record names the task tip that Finalize lands, and a trailer naming an earlier tip would rarely match the squash landing's tree, leaving the run attempted and unmapped.
- The walk is bounded by commit structure, not by the seed's `started_at`, so clock skew between machines cannot move the floor.

`internal/reportengine/doc.go` states the walk's floor.

## Not covered

- The report was not rerun against the hub's records; the regression row builds two darn runs over a parent sync trailer in the in-memory fake.
- A darn run whose time window holds no unclaimed parent commit still comes out unmapped, as before.
