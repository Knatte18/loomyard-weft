# Lens review

All four forks returned a report.

- criteria-coverage, MEDIUM, card 4 refuses bad-value global words D4 says to forward: fixed; `routeWords` now forwards `--json=bogus`, `--verbose=x` and `-h=bogus` to the last argv, the refusal and its error result are gone, and the row pins `orch start` running before `reed attach`'s root fails.
- criteria-coverage, LOW, `cmd/lyx/main.go`'s package doc and root `Long` do not name the shortcuts: fixed; card 5 updates both file comments and adds a Shortcuts line to the root `Long`.
- criteria-coverage, LOW, no row pins a held reading after the up line landed: fixed; card 2 adds a landed-up-line `dead` row.
- criteria-coverage, LOW, stdin wiring untested: fixed; card 4 adds a row where `reed attach` reads the shortcut's input.
- criteria-coverage, LOW, stencil-seed skip annotation is unrecorded scope: fixed; the annotation is dropped, and card 4 confirms the seeding repeat writes nothing because only a due subtree is written.
- row-reachability, MEDIUM, card 1 frame 4 cannot recover run-b while the rewrite reuses the failed modification time: fixed; the card requires a modification time other than `minutesAgo(40)`.
- row-reachability, LOW, `TestPasteSubmit` has no text field for the `-n run a;` row: fixed by dropping the row (see test-economy below).
- row-reachability, LOW, `-h=bogus` is forwarded and half a sequence runs: fixed together with the criteria-coverage MEDIUM; the card now states such words are forwarded, as D4 says.
- forced-edits, LOW, `cmd/lyx/main.go` file comments go stale: fixed, merged with the criteria-coverage finding above.
- forced-edits, LOW, card 1 frame 4 modification time: fixed, merged with the row-reachability MEDIUM.
- forced-edits, LOW, `execHook` comment wording: fixed; card 3 says the stdin bytes pass beside the subcommand argv.
- test-economy, LOW, `TestPasteSubmit`'s `-n run a;` row repeats coverage: fixed; the row is dropped, the integration test pins that text in a real pane.
- test-economy, LOW, card 5's `prime --json` step repeats card 4's rows: fixed; the step is dropped.
- test-economy, LOW, card 1's resumed-only frame re-tests `pauserecord`: fixed; the frame is dropped, `TestCovering` pins it.
- test-economy, LOW, `TestMount_File` re-asserts `Use`, `Short` and audience: rejected; the form walk checks only their shape, not that `Short` is built from the argvs or that `attach` gets its one-argv `Short`, which is this package's behavior.
