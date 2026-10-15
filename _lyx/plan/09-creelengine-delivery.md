# Card 9 — creelengine-delivery

**Create:**
- `internal/creelengine/deliver.go`
- `internal/creelengine/reminder.go`
- `internal/creelengine/sweep.go`
- `internal/creelengine/deliver_test.go`
- `plan:internal/creelengine#Scheduler` -> `type Scheduler struct`

**Edit:**
- `cmd/lyx#unboundedWaitFiles`
- `//cmd/lyx/testdata/test-budget.yaml`
- `//pattern/PATTERN-daemon-wakeups.md`

**Intent:** Deliver what the pool holds into panes through the seam, gated on the receiver being idle (D7, D9 delivery half, D10, D13, D14, D15, D16, D19).
`Scheduler` runs one worker per receiver with pending deliveries (queued injects, or unread mail not muted by a hold or the orch rule), at most one delivery in flight per receiver, so a receiver whose sends stay busy delays no other.
A worker probes `Delivery.Idle` from a 1-second base, doubling to 8 seconds while the receiver reads busy, held or paused, and returns to the base on a new post, inject or release; a receiver with nothing pending is not probed.
A probe for a dead or dormant receiver sends nothing and keeps its mail and queue; delivery resumes when `Strands` reports it live.
While `Delivery.Paused` covers the receiver nothing is typed, injects and reminders alike (D14).
Nothing is ever typed into an orch strand (`agentname.MatchesRole` on `orch`, so `orch-N` included); its reminders are journaled withheld naming `creel-notices` (D16).
Injects go first, in queue order: the worker appends and syncs `sending`, then calls `Send`; `nil` journals `injected`, `ErrBusy` retries at the next probe, `ErrNotLanded` journals `unconfirmed` and the inject is never retyped; another error is logged at Warn once per change of error and retried.
`inject --interrupt`'s queued head is typed at the next idle after `Interrupt` ran.
An inject's TTL runs only while the receiver is live and unpaused; a hold does not stop it, so an inject a hold mutes past its TTL expires; one still queued at expiry is journaled `expired` and never typed.
The consumed TTL is persisted as an `aged` event when the receiver turns live, dormant, paused or unpaused, every minute while the TTL runs, and when the scheduler stops.
`reminder.go` composes the reminder fresh from the unread set at delivery: `[creel] 2 unread: #41 from ly:x:driver «subject», #42 … — run lyx creel read`, at most 5 messages listed and the rest counted; a subject is cut to 80 runes with control characters and `«`/`»` replaced by spaces.
A reminder goes out at each busy-to-idle edge, except the first idle edge after creel's own reminder while the unread set is unchanged: then the next waits a backoff of 5 minutes, doubling to a 1-hour ceiling, reset by any change of the unread set.
A program sender's mailbox is never reminded.
While a hold covers the receiver, other senders' injects wait and their mail is not reminded; the hold owner's injects pass and its own unread mail is reminded alone; a hold ends at its expiry.
Both interrupt forms (D10), `interrupt` and `inject --interrupt`, run their pause and hold refusals here, after card 7's privilege and orch checks and before anything is queued or sent: refused while a pause covers the receiver; refused from any sender but the hold owner while a hold covers it, naming the owner and expiry; the hold owner's own interrupt passes.
Otherwise `Delivery.Interrupt` runs at once, bypassing the queue, and `ErrNoRunningTurn` is reported to the caller as nothing sent; `inject --interrupt` then queues its text at the head.
`sweep.go` (D15): while unread mail or a queued inject exists, a sweep every 30 seconds lists strands; a holder that is a strand name absent from two consecutive successful sweeps is gone; each unread message it holds goes one step back along its `Path` (the last forwarder, else `From`), journaled `returned` with the receiver that ended unread, and is reminded there; when that step is gone too it goes on toward the sender; when the sender is gone, EXTERNAL or a program, it is journaled `undelivered` and a `--wait-reply` waiter on it is answered returned-undelivered; each inject queued for a gone receiver is journaled `expired`, answering its `--wait`.
A sweep whose `Strands` call fails moves nothing and counts toward no streak, and a strand removed and re-added between sweeps keeps its mail.
Bound: an agent that never reads costs one reminder turn per backoff step at most.
`internal/creelengine/deliver.go` and `internal/creelengine/sweep.go` join `cmd/lyx#unboundedWaitFiles`, and `pattern/PATTERN-daemon-wakeups.md`'s What-counts list gains creel's delivery workers (1 second doubling to 8 seconds) and its removed-holder sweep (every 30 seconds while mail or injects exist).
`deliver_test.go`, every row on `creelfake`'s clock and delivery: `TestScheduler_Reminders` (a post to a live idle strand reminded once; the probe interval of a busy receiver read off the fake clock's waits as 1, 2, 4, 8 and 8 seconds, back to 1 second after a new post; a receiver with nothing pending never probed; a busy receiver reminded at its next idle edge; an unanswered reminder turn backs off 5 minutes, doubles to the 1-hour ceiling, and a new message resets it; 7 unread list 5 and count 2; a 100-rune subject with a newline and `»` cut to 80 runes with both replaced; a program mailbox never reminded; an orch and an `orch-2` strand never typed into and their reminders journaled withheld; a pause types nothing until lifted while post still stores; a hold mutes another sender's reminder, lets the owner's pass and ends at expiry; a dormant receiver keeps its mail and is reminded once live).
`TestScheduler_Injects` (injects typed before reminders in queue order; `ErrBusy` retried; `ErrNotLanded` journaled unconfirmed and never retyped; the TTL frozen while paused or dormant and running under another sender's hold until it expires untyped; an expired inject never typed; `inject --interrupt` interrupts then types its text first at the next idle; `interrupt` and `inject --interrupt` each refused under a pause and refused from a non-owner under a hold naming owner and expiry, with nothing queued, interrupted or typed; the hold owner's interrupt passing under its own hold; interrupt reported nothing-sent on `ErrNoRunningTurn`; a receiver whose sends stay busy delays no other receiver's delivery; a scripted send error other than the two sentinels, repeated over three probes and then changed, logged at Warn twice through `logcapture` and retried, which makes this test non-parallel).
`TestSweep_RemovedHolders` (mail returned one step after two sweeps and marked with the receiver that ended unread, then on toward the sender, then journaled undelivered with its waiter answered; a strand removed and re-added between sweeps keeps its mail; a failed strand listing moves nothing and resets no streak; an inject queued for a removed strand journaled expired, its `--wait` answered and absent from the next retention carry).
`cmd/lyx/testdata/test-budget.yaml` raises `internal/creelengine` by three.

**ImpactSummary:** New delivery files in creelengine, two files added to the daemon-wakeups scan list and its background, one budget raise; no existing behavior changes.
