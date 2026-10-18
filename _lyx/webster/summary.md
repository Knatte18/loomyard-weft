# Creel stage 2: notices and the orch's typed lines go through creel

All nine batches reached `status: done`.
No digest carried `departures` or warnings, and no batch needed recovery.
The scope guard listed `deviations` for every batch but 08; they are files changed outside a batch's own targets, and are informational.

- **01-shuttle-event-time** (head `5f0ad801b`, covers cards 1 and 2): shuttle events carry a hook-side time.
  Creel delivers into orch strands and refuses a hold on one unless the sender is a program.
  The reminder carries no subject.
  Deviations: `internal/creelengine/deliver_test.go`, `internal/shuttleengine/session.go`, `internal/shuttleengine/session_test.go`.
- **03-creel-command-inject** (head `18ba0a08a`): command injects from a program to an orch strand through a new `Delivery` operation.
  Deviations: `internal/creelcli/cli.go`, `internal/creelcli/delivery.go`, `internal/creelengine/command.go`, `internal/creelengine/deliver_test.go`, `internal/creelengine/journal_test.go`, `internal/creelengine/seam.go`.
- **04-creel-withdraw-and-refusal-code** (head `b85ca203e`, covers cards 4 and 5): a Go-only withdraw verb, the outside-pane refusal code, a bounded client call and the exported `GoClient`.
  Deviations: `internal/creelcli/cli.go`, `internal/creelengine/identity.go`, `internal/creelengine/protocol.go`.
  The fork reported a window where a withdraw can race an inject that is read but not yet marked sending; the card gave no way to close it.
- **06-orch-notify-over-creel** (head `5866bf5bd`): `NotifyPrime` posts creel mail, and every notice caller moves to it.
  Deviations: `internal/hibernatecli/cli.go`, `internal/orchcli/notify.go`, `internal/orchcli/notify_test.go`.
  The fork re-ran only `battencli` after its last fix, not the whole batch gate.
- **07-tender-orch-revive-inject** (head `401af3e8c`): tender's orch revive is a checked creel inject.
  Deviations: `internal/tendercli/deps.go`, `internal/tenderengine/pause_test.go`, `internal/tenderengine/resume.go`.
  The fork reported appending one test to `internal/tendercli/deps_test.go` with a shell heredoc instead of an Edit, which the edit rules forbid; the audit raised no warning.
- **08-drop-hub-notice-log** (head `5a5ee397d`): the hub notice log and its paths are removed, with both declared PATTERN changes made.
  No deviations.
- **09-orch-creel-seam** (head `32e24aaf4`): the watcher's creel seam, the pause-resume line as mail, the notice queue removed.
  Deviations: `internal/creelengine/fold.go`, `internal/orchcli/creelseam.go`, `internal/orchcli/fakecreel_test.go`, `internal/orchcli/watch_test.go`, `internal/orchengine/creel.go`, `internal/orchengine/fakecreel_test.go`, `internal/orchengine/pause.go`, `internal/orchengine/watcher.go`, `internal/orchengine/watcher_test.go`, `internal/orchengine/watchloop.go`.
- **10-orch-cycle-over-creel** (head `449dfed1f`): the watcher's cycle is delivered and confirmed through creel.
  Deviations: `internal/orchengine/fakecreel_test.go`, `internal/orchengine/prompt.go`, `internal/orchengine/watcher.go`, `internal/orchengine/watcher_test.go`, `internal/orchengine/watchloop.go`, `internal/orchengine/watchloop_test.go`.
  The fork compiled the llm tier but did not run it.
- **11-orch-start-withdraws** (head `cd4944e33`): `lyx orch start` runs two withdraw rounds, and the watcher retries a failed one.
  Deviations: `internal/orchcli/cli_test.go`, `internal/orchcli/start.go`, `internal/orchcli/start_test.go`, `internal/orchengine/watcher.go`.

The plan-level verify gate runs after this turn ends.
