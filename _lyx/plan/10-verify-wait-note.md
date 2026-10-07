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

**Intent:**
`lyx loom status`'s `waiting` field reports a running verify in the form `<producer>: <what> running <elapsed> (<detail>)` (D1), e.g. `Webster-Burler: verify running 6m (attempt 2; go test -tags integration ./...)`.

- `verifytree.Marker` gains `Command` (`yaml:"command"`), which `Verify` writes beside the site, attempt, start and pid.
  `Marker.Note` and its `since HH:MM` stamp go: the marker carries data, and loom renders the note, so the elapsed time comes from loom's injectable clock rather than a stamp.
  `marker_test.go` drops the note table and pins the command's round trip in `ReadMarker`; the dead-pid-reads-absent rule is unchanged.
- The producer name comes from the run's status: `shedverbs.Hooks.Waiting` becomes `func(st shedengine.Status) (string, error)`, and `waitingNote` passes the status it already read, for the one-shot envelope and the `--watch` line alike.
  `spec.go`'s field comment says the hook gets the status, so a product can name the current producer.
  `status_test.go`'s hook fakes take the new parameter, and one row pins that the hook receives the status file's `CurrentProducer`.
- `loomcli`'s `verifyWaiting` reads the marker and renders it with one formatter, `<producer>: verify[ <site>] running <elapsed> (attempt <n>; <command>)`, where the site is shown only when it differs from the producer name and the attempt clause is dropped when the marker's attempt is 0.
  Elapsed renders as whole seconds under a minute, whole minutes under an hour and hours with minutes beyond (`45s`, `6m`, `1h12m`), from a `now func() time.Time` the hook is built with, `time.Now` in production.
  A marker whose pid is dead reads as absent and the search falls through, as today; the parent-review note keeps its own form.
  `reviewstatus_test.go`'s verify table pins the rendered note under a fixed clock (site shown and hidden, attempt shown and dropped), the dead-marker fall-through and the empty result.

The formatter is the one card 12 reuses for shuttle's waits.

**ImpactSummary:** The `waiting` text of `lyx loom status` changes shape during a verify; the Waiting hook's one product (loom) and shedverbs' two call sites move to the new signature.
