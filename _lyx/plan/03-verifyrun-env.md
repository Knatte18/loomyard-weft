# Card 3 — verifyrun-env

**Edit:**
- `internal/verifyrun#Run` -> `func Run(ctx context.Context, command, dir string, env []string, out io.Writer) (int, error)`
- `internal/verifyrun#run` -> `func run(ctx context.Context, command, dir string, env []string, out io.Writer, delay time.Duration) (int, error)`
- `internal/verifytree#Verify`
- `internal/websterengine#rerunCardVerifies`
- `internal/verifyrun/run_integration_test.go`

**Intent:**
Give the one shell runner an environment parameter, so its callers can hand the verify shell the slot's `GOFLAGS` cap and inheritance variable (D4) while `verifyrun` itself stays free of the slot engine.
A nil `env` runs the shell with the parent's environment, as today; a non-nil one is the shell's whole environment (`exec.Cmd.Env`).
The package doc's import pin (standard library and `internal/logger`) stays as it is.
`verifytree.Verify` and `rerunCardVerifies` pass nil for now; later cards hand them a lease's environment.
`TestRun` gains a row whose command echoes a variable set only in the told environment and asserts it reaches the output, and every existing `Run` and `run` call in the test file passes nil.

**ImpactSummary:** Two callers outside the package, both passing nil, so no behaviour changes yet.
