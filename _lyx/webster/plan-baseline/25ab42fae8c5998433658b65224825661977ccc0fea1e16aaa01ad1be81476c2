# Card 10 — loom status names the producer, the elapsed time and the command of a running verify

**Edit:**
- `internal/verifytree/marker.go`
- `internal/verifytree/marker_test.go`
- `internal/verifytree/verify.go`
- `internal/shedverbs/spec.go`
- `internal/shedverbs/status.go`
- `internal/shedverbs/status_test.go`
- `internal/loomcli/reviewstatus.go`
- `internal/loomcli/reviewstatus_test.go`
- `contracts/specs/loom-status-spec.md`
- `docs/overview.md`

**Intent:**
`lyx loom status`'s `waiting` field reports a running verify in the form `<producer>: <what> running <elapsed> (<detail>)` (D1), e.g. `Webster-Burler: verify running 6m (attempt 2; go test -tags integration ./...)`.

- `verifytree.Marker` gains `Command` (`yaml:"command"`), which `Verify` writes beside the site, attempt, start and pid.
  `Marker.Note` and its `since HH:MM` stamp go: the marker carries data, and loom renders the note, so the elapsed time comes from loom's injectable clock rather than a stamp.
  `marker_test.go` drops the note table and pins the command's round trip in `ReadMarker`; the dead-pid-reads-absent rule is unchanged.
- The producer name comes from the run's status: `shedverbs.Hooks.Waiting` becomes `func(st shedengine.Status) (string, error)`, and `waitingNote` passes the status it already read, for the one-shot envelope and the `--watch` line alike.
  `spec.go`'s field comment says the hook gets the status, so a product can name the current producer.
  `status_test.go`'s hook fakes take the new parameter, and one row pins that the hook receives the status file's `CurrentProducer`.
- `loomcli`'s `verifyWaiting` reads the marker and renders it with one formatter, `<producer>: verify running <elapsed> (attempt <n>; <command>)`, where the attempt clause is dropped when the marker's attempt is 0.
  The note carries no site: the producer already names the site at every site a run waits at (`Webster`, `Webster-Burler`, `Publish`, `Finalize`), and `Marker.Site` stays in the marker.
  Elapsed renders as whole seconds under a minute, whole minutes under an hour and hours with minutes beyond (`45s`, `6m`, `1h12m`), from a `now func() time.Time` the hook is built with, `time.Now` in production.
  A marker whose pid is dead reads as absent and the search falls through, as today; the parent-review note keeps its own form.
  `reviewstatus_test.go`'s verify table pins the rendered note under a fixed clock (attempt shown and dropped), the dead-marker fall-through and the empty result.

The formatter is the one card 12 reuses for shuttle's waits.

`contracts/specs/loom-status-spec.md`'s section "The `waiting` key" is rewritten for the whole field (D1): the key appears while the run is `running` and any of three waits is present, read in order (a running verify, a shuttle wait that card 12 adds, the parent-review note), the first with a live pid winning;
its value is one line, `<producer>: <what> running <elapsed> (<detail>)` for the first two and the existing `parent review: ` form for the third;
the key stays absent when none applies or the run is in any other state, and it is still read from the markers and the round store, never from the status file.
`docs/overview.md`'s sentence that `status` carries a waiting-on-reviewer note is widened to the same three waits.

**ImpactSummary:** The `waiting` text of `lyx loom status` changes shape during a verify; the Waiting hook's one product (loom) and shedverbs' two call sites move to the new signature, and the status spec's `waiting` section and one overview sentence are rewritten to match.
