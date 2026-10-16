# Card 2 — refusal-stderr-fallback

**Edit:**
- `internal/creelcli#refuse`
- `internal/creelcli#runVerb`
- `internal/creelcli#readCmd`
- `internal/creelcli#runCreelInto` -> `func runCreelInto(cwd string, out, stderr io.Writer, args ...string) int`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli/doc.go`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`

**Create:**
- `internal/creelcli#writeRefusal`

**Uses:**
- `internal/creelcli#writeOk`
- `internal/creelcli#failingWriter`
- `internal/creelcli#lockedBuffer`
- `internal/creelcli#rig.startDaemon`

**Intent:** Fix #681 (D2): a creel refusal whose stdout write fails goes to stderr.

`writeRefusal` writes the JSON error envelope (`ok` false, `error` the trimmed message, the caller's fields) to `cmd.OutOrStdout()` with a checked write, the same shape `output.ErrFields` writes, and records the exit code with `clihelp.SetExit`.
When that write fails it writes one line `creel: <msg>` to `cmd.ErrOrStderr()`, and the exit code stays the one it was given.
`refuse` writes through it with exit 1, and `runVerb`'s two other envelopes with `ok` false, the wait timeout's and a body's non-zero result code, write through it too, so every creel verb's refusal has the fallback; `internal/output` is not changed.
`readCmd` wraps the error of a failed write of the messages so it names the write and says the messages stay unread, as in `creel read: could not write the messages, so they stay unread: <error>`, and the refusal carries that text to stderr when stdout is the writer that failed.
A stdout that is a closed pipe ends the process by SIGPIPE before any line; no acknowledgement was sent, so the mail stays unread, which needs no code.

Test: `runCreelInto` takes the writer stderr goes to in place of `io.Discard`.
The row "read into an output that cannot be written exits non-zero and acknowledges nothing" of `TestCreelCLI` runs both of its reads into a `failingWriter` with a `lockedBuffer` as stderr, and asserts for each exit code exactly 1 and a stderr line starting `creel: `; for the read of a message it also asserts the line holds the write's error and `unread`, and its existing retry still reads the message back.
The read of nothing unread reaches the refusal of a failed write of the empty envelope, and the read of a message reaches the wrapped write error, after every upstream check passed.
A new row "a waiting verb's envelope that cannot be written keeps its exit code and goes to stderr" reaches `runVerb`'s two other envelopes over the rig's daemon, each run into a `failingWriter` with a `lockedBuffer` as stderr: an `inject --wait` whose send answers `ErrNotLanded` exits `ExitInjectUnconfirmed` with a `creel: ` line naming the unconfirmed landing, and an `inject --wait --timeout 5s` into a never-idle strand, its client clock advanced 5 seconds, exits `ExitWaitTimeout` with a `creel: ` line naming the timeout.

Docs in the same commit: `internal/creelcli/doc.go` says a refusal whose JSON envelope cannot be written to stdout writes `creel: <error>` to stderr with the same exit code; the C4 watch line of `tools/sandbox/SANDBOX-CREEL-SUITE.md` says `lyx creel read > /dev/full` exits 1, writes a `creel:` line to stderr and leaves the mail unread.

**ImpactSummary:** The refusal and envelope writes of every `lyx creel` verb go through one checked helper in `internal/creelcli`; stdout output is unchanged when it can be written, and the only other change is one test helper's added stderr parameter with its one caller row.
