# Card 2 — gitexec classifies its own transport failures

**Create:**
- `plan:internal/gitexec#IsTransportFailure` -> `func IsTransportFailure(err error) bool`
- `internal/gitexec/transport.go`
- `internal/gitexec/transport_test.go`
- `internal/gitexec/transport_integration_test.go`

**Uses:**
- `internal/gitexec#GitError`

**Intent:**
`internal/gitexec` exposes its own classification of a git remote operation that failed to reach the remote (decision record D2).
The translation into `internal/shedengine`'s mark (card 5) then never text-matches outside the package that owns the failing call, and `gitexec` gains no `shedengine` import.

`IsTransportFailure` (`internal/gitexec/transport.go`) reports whether `err`'s chain holds a `*GitError` for a remote operation that failed to reach the remote.
Both conditions must hold:
- The git subcommand in `Args` is `push`, `fetch`, `ls-remote` or `clone`, D2's four remote operations and no others.
  The subcommand is the first argument after git's global options: `-c` and `-C` each consume the argument that follows them, and any other argument starting with `-` is skipped.
  This matters because `internal/gitrepo` runs every push as `-c push.autoSetupRemote=true push`, so a plain "first non-flag argument" rule would read `push.autoSetupRemote=true` as the subcommand.
- The lower-cased `Stderr` contains one of a closed set of transport phrases, declared once as an unexported slice: `could not resolve host`, `connection refused`, `connection reset`, `connection timed out`, `operation timed out`, `network is unreachable`, `failed to connect to`.
Anything else is not a transport failure: a rejected or non-fast-forward push, an authentication failure, a missing ref, a local command, and an error with no `*GitError` in its chain.
The doc comment states that an unrecognised message format falls back to "not transient", today's behaviour.

Tests:
- `transport_test.go` is untagged and spawns nothing; it builds `*GitError` values directly.
  - True: a `push` and a `fetch` with each transport phrase, also when wrapped with `%w`; `-c push.autoSetupRemote=true push` with `Failed to connect to`; `-C <dir> fetch` with `Could not resolve host`.
  - False: a `push` rejected as non-fast-forward, the same under `-c push.autoSetupRemote=true`, a `push` failing authentication, a `fetch` of a missing ref, a `pull` and a `status` whose stderr happens to contain a transport phrase, a plain `errors.New`, and nil.
- `transport_integration_test.go` (`//go:build integration`, under the package's existing hermetic `TestMain`) pins the real stderr.
  It commits once in a temporary repository, sets `origin` to `http://127.0.0.1:1/unreachable.git`, and runs `Run([]string{"-c", "push.autoSetupRemote=true", "push", "origin", "HEAD"}, dir)`.
  The returned error satisfies `IsTransportFailure`.
  A second case pushes to a local bare repository whose `pre-receive` hook rejects every push, and asserts false.

**Commit:** 2: gitexec: classify transport failures
