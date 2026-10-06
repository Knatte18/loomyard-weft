# Card 2 — exclusion-out-of-process

**Edit:**
- `cmd/testtiming/spawnscan.go`
- `cmd/testtiming/spawnscan_test.go`
- `cmd/testtiming/redundancy.go`
- `cmd/testtiming/redundancy_test.go`
- `cmd/testtiming/main.go`
- `pattern/PATTERN-test-economy.md`
- `docs/benchmarks/running-tests.md`

**Intent:**
D1: the redundancy tool excludes a test from judgment only when it may run this module's code in another process.

Today `spawnRef` treats `os/exec`, `internal/hubforge`, every `gitkit` export but `HermeticGitEnv`, every `tmuxkit` export but `Main` and any testkit function that reaches one of those as a spawn, and `measurePackage` marks a test the scan did not find as unresolved.
Replace that rule with the decision record's:

- A test is excluded as out of process when, directly or through a followed same-package or testkit call, it references `internal/testkit/lyxbin`, `os.Executable`, `os.Args[0]` (an index expression on `os.Args` with the literal index `0`, so a read of `os.Args[1:]` alone does not count), or an `exec.Command`/`exec.CommandContext` call whose program argument is the string literal `"go"`.
  `os/exec` in general, `hubforge`, `gitkit` and `tmuxkit` exports no longer count.
  The testkit walk (`kitSpawns`) applies the same rule to kit sources, so a kit that only spawns git stops counting.
- A test declared in a `tmux`-tier file is excluded as out of process, because reed panes run `lyx`.
  The tier is read from the declaring file's `//go:build` line with `go/build/constraint`: a file is `tmux`-tier when its expression mentions the `tmux` tag (`tmux && linux`, `tmux && !windows` count), and tier-tagged when it mentions `integration`, `tmux` or `llm`.
  `loadPkgSource` records the file's tier on each `funcInfo`.
- A test in a file that is not tier-tagged is never excluded as out of process or unresolved, whatever the scan finds, because `PATTERN-test-speed` forbids an untagged test to spawn.
- A test in a tier-tagged file with a call the scan cannot resolve stays excluded as `unclassifiable call`, as today.

Rename the out-of-process reason: `reasonSpawns` becomes a reason whose text says the test runs module code out of process (for example `out of process`), and `spawnVerdict`'s field names follow the new meaning.
`noCoverageReason` and `classify` keep their structure.

Bound: the change only moves tests from the "no coverage" list into judgment; it never deletes or edits a test, and every new candidate is still read by hand before deletion (D4).
The scan does not see a hook, alias or helper a test installs so that git runs module code; the docs say so and name D4's hand check as the backstop.

Tests: rework `TestScanSpawns`'s fixture so its rows cover each rule — a `lyxbin` reference, `os.Executable`, `os.Args[0]`, a read of `os.Args[1:]` that stays judged, `exec.Command("go", …)`, a direct `exec.Command("git", …)` that is no longer excluded, a gitkit spawn and a hubforge call that are no longer excluded, a kit that spawns only git, a test in a `tmux` file and one in a `tmux && linux` file, an unresolved call in an `integration` file (excluded) and in an untagged file (judged), and a `lyxbin` reference in an untagged file (judged).
`TestClassify`'s no-coverage row keeps covering the renamed reason.

Docs, same commit: the package doc in `main.go` describes the new exclusion rule in place of "spawning a subprocess"; `pattern/PATTERN-test-economy.md` states what the "no coverage" list now means and that D4's hand check also asks whether a candidate makes git run module code; `docs/benchmarks/running-tests.md`'s `cmd/testtiming` section gains the rule in a sentence or two, pointing at the PATTERN background rather than restating it.

**ImpactSummary:** Narrows the redundancy mode's exclusion rule and its docs; no other package imports `cmd/testtiming`, and the timing mode is untouched.
