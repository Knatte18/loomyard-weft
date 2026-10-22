# darn-verify-fixes: darn failures route to Merge-In when Darn-Write cannot own them

All four batches reached `status: done`, with no departures reported.

- **01-impactset-cmd-lyx-join** (head `1d3e7f86a`): the impacted set joins binary-driving test packages only for a change to `cmd/lyx`, and lists them on `Targets`.
  Merge-In and Publish take the record's target and the reason's way forward from a told classifier.
  A darn failure is classified as owned or not owned, and a not-owned one routes to Merge-In.
  Batten's retry of a Merge-In target is pinned, and the gate outcome carries the failing entry's findings text.
  Deviations: `internal/battenshed/innerrun_publishretry_test.go`, `internal/loomcli/darnclassify.go`.
- **06-shedrun-gate-findings-file** (head `1a09b16b7`): the per-row gate findings file is declared under the run's scratch directory, gated producers write, name and clear it, and every gated row is told its path.
  Deviations: `internal/shedadapters/burlerfix.go`, `burlerfix_test.go`, `burlerreview.go`, `burlerreview_test.go`, `gateattempts.go`, `gateattempts_test.go`, `internal/shedrecipe/entries_burlerhalves_test.go`, `entries_discussionwrite_test.go`, `env.go`, `multillm_fixture_test.go`, and `internal/shedrun/paths.go`.
- **09-darn-write-reroute-edge** (head `e9f0e6190`): Darn-Write has an `on_reroute: Merge-In` edge, and the prior-work note skips reroutes.
  No deviations.
- **10-darn-write-entry-reroute** (head `4f38202ac`): Darn-Write reroutes to Merge-In on this run's failure, an unmoved branch, a clean tree and a moved parent.
  The shell guard lets the tmux-tier reproduce command through every row.
  Deviation: `internal/loomshed/darnwrite_test.go`.

The scope guard reported only the deviations listed above, all informational.
