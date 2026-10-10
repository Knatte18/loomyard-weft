# Card 12 — orch-resume-context-verb

**Create:**
- `internal/orchcli#orchCLI.resumeContextCmd`
- `internal/orchcli#resumeContextCommand`

**Edit:**
- `internal/orchcli#Command`
- `internal/orchcli#orchCLI.orchSpec`
- `internal/orchcli#orchPaths`
- `internal/orchcli/cli_test.go`
- `internal/orchcli/start_test.go`
- `internal/orchcli/doc.go`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/orchengine#ResumeContext`
- `internal/shuttleengine/claudeengine#SessionStartContext`
- `internal/shell#Shell`

**Intent:** The CLI half of D11: the hidden `lyx orch resume-context` verb and the hook command the orch run's spec carries.
`resumeContextCmd` is a hidden cobra verb under the orch subtree with a `Short`, registered in `Command` beside `watch`; it runs `orchengine.ResumeContext(c.paths, c.stencilsDir, time.Now())` and writes `claudeengine.SessionStartContext(text)` to the command's output and nothing else, so its standard output is the provider's additional-context JSON; any failure prints the JSON error envelope through `output.Err` and writes no mark, and the typed pointer then delivers it.
`orchPaths` fills `ResumeMarkPath` with a file beside `cycle-request`.
`resumeContextCommand` builds the hook command through the POSIX dialect, which Claude Code's hooks run under on every OS (git-bash on Windows): `shell.Posix().Chain(sh.ChangeDir(primeAnchor), "lyx orch resume-context")`, so the verb runs in the prime whatever cwd the orch session has moved to; `lyx` is named bare, which resolves to the binary that spawned the session through the pane's `panebin.go` prelude on `PATH`.
`orchSpec` sets `ContextAfterCompaction` to that command for the prime's anchor, so the orch run is the one spec that installs the hook.
Bound: the hook injects context into the orch session at every compaction; it is bounded to the orch run and to the rendered pointer text, which asks the session to read the role and the note and executes nothing, and a verb failure shows as an error in Claude Code and adds no context.
`start_test.go` asserts the started spec carries the command naming the prime's anchor and the verb; `cli_test.go` drives the verb over a seeded state and stencils: it prints the hook JSON with the expected pointer in each phase, refuses outside the prime and on a missing stencil with a JSON error and no mark (the absent-state refusal is `ResumeContext`'s, tested with it), and the `TestCLITree_EveryCommand` walk keeps passing since the verb carries a `Short`.
`orchcli/doc.go` and `orchengine/doc.go`'s verb list name the hidden verb and its hook.

**ImpactSummary:** One hidden verb and one spec field set at the one orch launch site; no other verb or spec changes.
