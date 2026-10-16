# Card 13 — daemon-log-lines

**Create:**
- `plan:internal/creelcli#pidLineWriter` -> `type pidLineWriter struct`
- `plan:internal/creelcli#pidLineWriter.Write` -> `func (w *pidLineWriter) Write(p []byte) (int, error)`

**Edit:**
- `internal/creelcli#daemonCmd`
- `internal/creelcli#runDaemon` -> `func runDaemon(ctx context.Context, hubPath string, stderr io.Writer) error`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli/doc.go`

**Uses:**
- `plan:internal/creelengine#DaemonStartHeader`
- `plan:internal/creelengine#DaemonLogPrefix`
- `internal/creelengine#Geometry`
- `plan:internal/creelcli#runCreelSplit`
- `plan:internal/creelcli#lockedBuffer`
- `internal/creelcli#newRig`
- `internal/testkit/logcapture#Capture`

**Intent:** Decision D5, daemon half: the daemon writes its start header and pid-tagged warnings to the stderr the spawn pointed at the log file, so a start failure reaches the file.

`daemonCmd` hands `runDaemon` `cmd.ErrOrStderr()`, which in the detached process is the log file card 12 opened.
`runDaemon` first writes `DaemonStartHeader(os.Getpid(), time.Now())` and a newline to it, then points the logger's stderr half at a `pidLineWriter` over it with `logger.SetOutput` and sets the stderr threshold to Warn with `logger.SetVerbosity(0)`, so the daemon's delivery and journal warnings (`worker.report`, the journal's torn-line warning, the poll's roll failure) reach the file, each once per change as today.
`pidLineWriter` opens every line it writes with `DaemonLogPrefix(os.Getpid())`, also every line of a write holding several, and buffers a line until its newline, so two daemons started at once, whose lines interleave in the one file, stay apart.
A start that fails (`ErrDaemonRunning` aside) is logged at Warn through the logger before `daemonCmd` refuses, so its error line, such as the journal's malformed-line error naming the file and line, lands in the file with its pid; a runtime panic prints its trace to the same stderr with no prefix, which the client's excerpt quotes after the header.
Bound: the file grows only by start headers, start errors and warnings, each warning logged once per change, and card 12's cap bounds it.

Tests, rows of `TestCreelCLI`, which capture the process-global logger with `logcapture` and restore it:
a `pidLineWriter` handed one write of two lines and then a line in two writes prefixes all three lines with the pid, each once;
and `creel daemon --hub-path <hub>` over a journal whose middle line is malformed, run through `runCreelSplit` with a `lockedBuffer` as its stderr, exits non-zero with stderr holding the start header for the test process's pid followed by a pid-tagged line naming the journal file and line.
The existing row "daemon beside a live daemon exits 0 with its message" now runs `runDaemon`, which redirects the process-global logger, so it captures the logger with `logcapture` too and runs through `runCreelSplit`, keeping its exit-0 and message assertions on stdout while the start header goes to its stderr.
The `pidLineWriter` row stays a direct row: a line split across two writes is buffering no verb run can time, so only a direct write reaches it.

`internal/creelcli/doc.go` says the daemon writes its start header and pid-tagged warnings to its stderr, which the spawn points at the daemon log.

**ImpactSummary:** Changes only the hidden daemon verb and its unexported runner, whose one caller is daemonCmd; the operator verbs are untouched.
