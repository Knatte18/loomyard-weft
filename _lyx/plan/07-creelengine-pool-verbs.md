# Card 7 — creelengine-pool-verbs

**Create:**
- `internal/creelengine/seam.go`
- `internal/creelengine/pool.go`
- `internal/creelengine/verbs.go`
- `internal/creelengine/verbs_test.go`
- `internal/testkit/creelfake/creelfake.go`
- `plan:internal/creelengine#Strand` -> `type Strand struct`
- `plan:internal/creelengine#Delivery` -> `type Delivery interface`
- `plan:internal/creelengine#Sender` -> `type Sender struct`
- `plan:internal/creelengine#Store` -> `type Store struct`
- `plan:internal/testkit/creelfake#Delivery` -> `type Delivery struct`
- `plan:internal/testkit/creelfake#Clock` -> `type Clock struct`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Build the pool's verb logic over the journal: post, reply, read, inbox, forward, log, inject queueing, the interrupt verb's checks, holds and the privilege rules (D5, D7 storage half, D8, D9, D10 privilege half, D11, D12, D13, D16, D18).
`seam.go` declares the delivery seam (D1): `Strand` is one hub strand (`Name`, `GUID`, `Worktree`, `Slug`, `Live`, `PanePID`), and `Delivery` lists the hub's strands (`Strands() ([]Strand, error)`, an error naming what could not be read: the worktree list or a reed state), says whether a strand is idle (`Idle`) and whether a pause covers it (`Paused`), types one line (`Send`) and sends the guarded interrupt (`Interrupt`).
It declares the sentinels the adapter maps shuttle's answers to: `ErrBusy` (held, nothing typed, retry), `ErrNotLanded` (typed, not confirmed) and `ErrNoRunningTurn`; `creelengine` imports neither `reedengine` nor a concrete shuttle engine and names no tmux command.
`Sender` is a resolved caller: its full name, whether it is a strand, a program or EXTERNAL, and for EXTERNAL the envelope creel sees (terminal, parent process name, working directory), rendered `from ly:EXTERNAL (claude, pts/4, ~/Code/…)`.
`Store` holds the folded pool, the journal writer and the clock behind one mutex; every mutating verb appends and syncs its journal line before it changes the pool and before it answers (D3).
Receivers (D5): a full strand name from `Delivery.Strands`, or a program sender's name once it has posted or read; an unknown name is refused listing the names that exist; `<shortname>:EXTERNAL` as a receiver is refused naming `--wait-reply` and `--reply`; a `Strands` error is returned naming what could not be read, never as an unknown name.
`Post`: body non-empty and at most 64 KiB, else refused (the over-limit refusal names writing a file and mailing its path); stores the message for its receiver, answers its id, and wakes the receiver's delivery worker (card 9).
`Reply(sender, id, text)`: addresses the original message's `From`; the caller must hold or have read the message, else refused; an id the pool no longer holds is refused; to a strand or program it is mail; to an EXTERNAL sender it is handed to a waiter registered on that id (the registration is card 10's protocol), else journaled `undelivered`.
`Read(sender, from, all)`: `read` is the caller's unread mail, `--from` filters by sender, `--all` adds read messages still retained, oldest first with envelope; it reserves the unread ones it hands out so a concurrent read by the same holder skips them; `Ack` marks them read and journals `read`; `Release` drops the reservation and leaves them unread.
A program sender's first read makes its mailbox known, as its first post does.
`Inbox`: the caller's senders with unread counts; for an orch caller (`agentname.MatchesRole(role, agentname.RoleOrch)`, so `orch-N` included) the answer also states that its reminders are withheld until `creel-notices` (D16).
`read` and `inbox` from EXTERNAL are refused naming `--wait-reply`.
`Forward(sender, id, receiver)`: holder changes, `From` stays, the caller is appended to `Path`; refused for a message the caller does not hold, one it has read, a forward to itself, to EXTERNAL or to an unknown name.
`Log(filter)`: events filtered by id, from, to and since, with ids, parties, times and subjects and never a body; any caller may run it; a reminder to an orch shows as withheld naming `creel-notices` (D16).
`Inject(sender, receiver, text, ttl, interrupt)`: a text with a control character is refused; `ttl` defaults to 10 minutes and above 1 hour is refused; the inject is queued per receiver in order, or at the head when `interrupt` is set; card 9 adds the pause and hold refusals of the `interrupt` form and its interrupt key.
`Interrupt(sender, receiver)`: the privilege and orch checks below run here before card 9's pause and hold refusals and the guarded interrupt.
Privilege (D9): EXTERNAL, a program sender and an orch strand (`agentname.MatchesRole(role, agentname.RoleOrch)`, so `orch-N` included) are privileged; a non-privileged sender's inject, interrupt or hold on another strand is refused naming `lyx creel post`; any sender but an orch may inject into its own strand.
Orch receivers (D16): inject and interrupt into an orch strand are refused naming `lyx orch distill`, `lyx orch refresh` and the orch's own pane; hold and release on it are accepted and journaled.
`Hold(sender, receiver, for)`: privilege as D9, `for` above 1 hour refused, a hold on a receiver another sender holds refused naming the hold owner and its expiry, a repeat by the owner renews it; `Release` by the owner ends it and journals `released`; `Release` by another sender is refused naming the owner and expiry, and `Release` on a receiver with no live hold answers that nothing was held and journals nothing.
Bound: holding again renews with no overall cap, so a live privileged owner can mute a receiver while it keeps renewing; an owner that dies mutes it for at most 1 hour.
`internal/testkit/creelfake` (`PATTERN-testkit`, since `internal/creelcli`'s tests use it too) holds `Delivery`, a fake of the seam (scripted strands, idle and pause answers per strand, recorded sends and interrupts, scripted send and interrupt errors, a strand-list error), and `Clock`, a fake clock that advances only when told; it imports only `internal/creelengine` and the standard library.
`verbs_test.go`, over `creelfake`: `TestStore_Mail` rows for post (stored and acknowledged with its id, empty and over-64-KiB body refused, unknown receiver refused listing names, EXTERNAL receiver refused naming `--wait-reply`, a `Strands` error failing naming the unreadable part, a program mailbox known after its first post and, in another row, after its first read), read/read --from/read --all/inbox sets, an orch and an `orch-2` caller's inbox stating the withheld reminders, nothing unread answering an empty set, two concurrent reads by one holder never sharing a message, an un-acked read released leaving messages unread, EXTERNAL read and inbox refused, forward (holder and path change, `From` kept; not held, read, to itself, to EXTERNAL and unknown refused), reply (to a strand is mail, with no waiter journaled undelivered, unknown id refused, a caller that neither holds nor has read the message refused), and log filters by id, from, to and since with no body in any line and an orch reminder shown withheld.
`TestStore_InjectAndHold` rows: privileged and non-privileged inject into another strand (refusal names `lyx creel post`), a non-privileged interrupt into another strand refused naming `lyx creel post`, self-inject passing, an orch's and an `orch-2`'s self-inject refused, inject and interrupt into an orch and into an `orch-2` refused naming the way forward, control character refused, an inject with no `--ttl` carrying 10 minutes, `--ttl` above 1 hour refused, hold above 1 hour refused, a non-privileged hold refused naming `lyx creel post`, a second sender's hold refused naming owner and expiry, the owner's renew, release by a non-owner refused naming owner and expiry, release with no hold answering nothing held, hold and release on an orch accepted and journaled.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/creelengine` by two.

**ImpactSummary:** New files in the new creelengine package, a new test kit and one budget row raise; no existing caller.
