# Card 1 — testtiming reports per-package test count and serial time

**Edit:**
- `cmd/testtiming#pkgResult`
- `cmd/testtiming#parseLine`
- `cmd/testtiming#printReport`
- `cmd/testtiming#run`
- `cmd/testtiming#main`

**Create:**
- `cmd/testtiming/main_test.go`

**Intent:**
`cmd/testtiming`'s package table gains two columns per package: the top-level test count and the serial time, the sum of its top-level tests' elapsed seconds (D1).
Subtests stay out of both, the same rule the slowest-tests list already applies.
The command line `-full` stays and keeps meaning `-tags integration`;
a new `-tags` string flag names the build tags outright (`-tags integration,tmux`), and passing both is refused with an error naming the two flags.
The printed header line shows the exact `go test` command run, tags included, so a captured table records its own tier.
The package doc comment gains the new columns and flag.

`main_test.go` is untagged and spawns nothing: one table-driven test feeds a canned `go test -json` event stream (top-level passes, a subtest, a skip, a failing test, a `[no test files]` package) through the parse-and-aggregate path and asserts each package's exact count, serial time, elapsed and terminal action;
a second row set covers the `-full`/`-tags` resolution, including the refusal.
It calls `t.Parallel`.

**ImpactSummary:** confined to `cmd/testtiming`'s `main` package, a dev tool with no importers; its existing output gains columns and its flags gain `-tags`.
