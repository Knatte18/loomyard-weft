# Card 4 — creel-withdraw-and-refusal-code

**Create:**
- `plan:internal/creelengine#VerbWithdraw` -> `const VerbWithdraw = "withdraw"`
- `plan:internal/creelengine#WithdrawnMessage` -> `type WithdrawnMessage struct`
- `plan:internal/creelengine#WithdrawResult` -> `type WithdrawResult struct`
- `plan:internal/creelengine#Store.Withdraw` -> `func (s *Store) Withdraw(sender Sender, receiver string) (WithdrawResult, error)`
- `plan:internal/creelengine#WithdrawnReason` -> `const WithdrawnReason = "withdrawn"`
- `plan:internal/creelengine#CodeStrandClaimOutsidePane` -> `const CodeStrandClaimOutsidePane = "strand-claim-outside-pane"`
- `plan:internal/creelengine#ErrStrandClaimOutsidePane` -> `var ErrStrandClaimOutsidePane = errors.New("creel: the strand claim comes from a process in no strand pane")`

**Edit:**
- `internal/creelengine#Response`
- `internal/creelengine#resolveSender`
- `internal/creelengine#daemon.fail`
- `internal/creelengine#daemon.dispatch`
- `internal/creelengine#Client.Call`
- `internal/creelengine#Client.CallWatching`
- `internal/creelengine#Client.exchange`
- `internal/creelcli#postCmd`
- `internal/creelcli#readCmd`
- `internal/creelengine#TestResolveSender`
- `internal/creelengine_test#TestStore_InjectAndHold`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelengine/doc.go`

**Intent:**
Two Go-client-only additions to creel, and a bound on every Go client call (decision record Scope, D5, D6, D7).

Withdraw (D5):
`Store.Withdraw` lets a program sender remove its own queued injects and its own unread mail to one receiver.
It refuses a sender that is not a program, naming that only a program may withdraw; input reaching it: a strand or EXTERNAL client sending `VerbWithdraw`, which the daemon resolves and dispatches.
It acts on the receiver by name whether or not creel still lists that strand, and answers an empty result when nothing of the caller's is queued there.
Each queued inject of the caller to the receiver is journaled `expired` with reason `withdrawn` and its waiters answered expired; each unread, unreserved message from the caller held by the receiver is journaled `undelivered` with reason `withdrawn` and a wait on it answered returned.
An inject already journaled `sending` and a message the receiver already read or is reading are left alone.
`WithdrawResult` answers the ids of the withdrawn injects and, as `WithdrawnMessage`, the id and subject of each withdrawn message; `Response` gains a `Withdrawn` field carrying it, and `daemon.dispatch` serves `VerbWithdraw`.
The CLI gains no verb: only Go code with a program claim reaches it.
Bound: it touches only the caller's own queued injects and unread mail on one receiver.

Typed refusal code (D6):
`resolveSender`'s refusal of a strand claim naming a strand creel lists, from a process in no strand pane, wraps `ErrStrandClaimOutsidePane`; it is the only refusal that does.
`daemon.fail` sets `Response.Code` to `CodeStrandClaimOutsidePane` when the error wraps it, and the Go client returns a refusal carrying that code as an error that `errors.Is` matches against `ErrStrandClaimOutsidePane` and whose text is the daemon's message.
A claim from another strand's pane, a program claim from a non-orch pane and the refusal while a pane process is unknown carry no code.

Call timeout (D7's "the Go client's call timeout"):
`Client.Call` and `CallWatching` apply their timeout to every request, not only a waiting one, so a non-waiting call to a daemon that accepts and never answers returns `ErrWaitTimeout` once the timeout passes; zero still means none.
`postCmd` and `readCmd` pass their `--timeout` only when they wait (`--wait-reply`, `--wait`) and zero otherwise, so a CLI call's behavior is unchanged.

Tests:
`TestResolveSender` gains rows: the outside-pane refusal wraps `ErrStrandClaimOutsidePane`, and the other-pane refusal and the unknown-pane-process refusal do not.
`TestStore_InjectAndHold` gains rows: a program's withdraw removes its queued injects and unread mail to `ly:orch`, journaled with reason `withdrawn`, answers their ids and subjects, and leaves another sender's inject and a read message; a registered `InjectWait` on a withdrawn inject is answered expired and a `PostWait` on a withdrawn message is answered returned; a withdraw on a name creel no longer lists with nothing queued answers empty; a strand's withdraw is refused.
`TestScheduler_Injects` gains a row: with the scheduler started and a send held open by the fake delivery's `SetSendFunc`, a withdraw leaves the inject journaled `sending` alone and it settles `injected`.
`TestDaemon_Protocol` gains rows: a program client's withdraw is answered with the result; a client whose strand claim names a listed strand from outside every pane gets an error that `errors.Is` matches; a non-waiting call to a listener that accepts and never answers returns `ErrWaitTimeout` at the timeout.
`internal/creelengine/doc.go` describes withdraw, the refusal code and the call timeout.

**ImpactSummary:** One new daemon verb and one response field, a typed wrap of one existing refusal, and a client timeout every CLI caller passes as zero; no existing verb changes behavior.
