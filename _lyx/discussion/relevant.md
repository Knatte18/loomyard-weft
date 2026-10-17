# Relevant code

- `internal/creelcli/cli.go` `runVerb`, `writeRefusal`, `writeOk`, `result`, `postCmd`, `injectCmd`, `readCmd`, `logCmd` — the success path, the stderr fallback and the verb bodies that store before answering
- `internal/creelcli/doc.go` — package doc that states the fallback line and gains the lost-answer line
- `internal/creelcli/cli_test.go` `failingWriter`, `runCreelInto`, `TestCreelCLI` — the failing-writer subtests the new cases extend
- `internal/creelengine/verbs.go` `Store.Log`, `LogFilter` — the id filter D4 widens
- `internal/creelengine/fold.go` `LogEntry`, `pool.logEntryFor` — carries `Unread` on reminded entries
- `internal/creelengine/doc.go` — the `# Log` section D5 updates
- `internal/creelengine/deliver_test.go` `eventsOf` — exact event-list assertions on `LogFilter{ID}` that D4 may change
- `internal/output/output.go` `Ok` — the shared helper whose dropped write error causes #699
- `internal/creelengine/client.go` `Client.Call`, `Client.exchange` — a wait timeout answers an empty response, so no id reaches the CLI
- `tools/sandbox/SANDBOX-CREEL-SUITE.md` — its `log --id` expectation gains the reminded events
