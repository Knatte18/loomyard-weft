# Support log — shuttle-start-hardening

## Interview

A darn run holds no interview: the writer read the board entry, the code and the board, and made every call itself.

## Rejected alternatives

- Passing the system-prompt notice through `--append-system-prompt-file`: the launch line already reaches the pane through reed's launch script, `internal/shell` quotes embedded quotes correctly, and the file flag is documented for print mode only.
- Retrying EAGAIN inside shuttle's liveness and removal callers: each caller would need its own loop, and capture and send would stay unguarded; the exec seam covers them all at once.
- Retrying the start in the shed driver instead of shuttle: the driver already re-steps a transient failure once, but that re-runs the whole producer and its preflight; one launch hiccup is cheaper to absorb where the launch happens.
- Retrying a start whose strand removal failed: the old strand may hold a live agent, so a second launch could leave two agents for one start.
- Recording a new terminal outcome such as `unverified` for an abandoned attempt: the outcome set is closed and read by attach, stop and the completion-signal scan, and the attempt is torn down either way.
- Withholding the notice for every turn the operator typed: shuttle cannot tell an operator's typed text from a parent's SendMessage answer, so it would silence a real follow-up question after the parent answers.
- Withholding the notice on any engine: an engine whose sessions do not wake on background completion stays idle after such a turn and needs the parent.

## Review rounds

_No rounds yet._

## Question ledger

- Is the long inline notice the cause of #870? Self-picked no: the launch script carries the line, and quoting is correct; the reported runs' captures are gone, so the start retry and the head of the capture cover a recurrence and show its first error line.
- Do #869 and #870 share a cause? Open: no capture survives; both are covered by the EAGAIN retry and the start retry.
- How many extra launches? Self-picked one, as a `shuttle.yaml` key so a hub can turn it off or raise it.
- Does the operator-typed clause need its own signal? Self-picked no: both reported cases had a background shell running, which the background rule covers.
