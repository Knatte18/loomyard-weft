# Relevant files

- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` — holds S9 and C1 to C13 today, and the preamble the creel suite adapts
- `tools/sandbox/suite.go` `suiteSpec`, `shuttleSuite`, `runSuite` — the embedded suite specs the creel spec joins
- `tools/sandbox/main.go` `run` — the subcommand dispatch the `creel-suite` case joins
- `tools/sandbox/suite_test.go` `TestRunSuite_PerSuiteSpec`, `TestSuiteSpecs_ReedTeardownFlag` — per-suite tables to extend
- `tools/sandbox/main_test.go` `TestRun_SuiteSubcommands` — subcommand table to extend
- `sandbox/posix/shuttle-suite.sh` — template for the POSIX creel launcher
- `sandbox/win/shuttle-suite.cmd` — template for the Windows creel launcher
- `cmd/lyx/sandbox_coverage_test.go` `parseCoveredModules` — globs every suite file, so the move needs no change here
- `docs/sandbox-howto.md` — names the suites and launchers
- `internal/creelengine/doc.go` — the daemon stops only on a replaced binary
