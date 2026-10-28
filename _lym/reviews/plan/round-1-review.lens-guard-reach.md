```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch and test row the cards add, I traced whether a real entry point reaches it and whether the guarded site comes up again on a path the card leaves unguarded.

**Per guard, reachable or not**

- Card 1, `Asker` prime and beam branches: reachable, through `TestAsker` rows on the prime and on `b`. The pair branch is reachable through row `m`. The origin read error is a propagated I/O error, so it is exempt.
- Card 1, `Asker` answering not recorded: reachable, on a hub with no orch state at the resolved worktree.
- Card 2, `--parent` with a receiver argument: reachable (`post --parent r t`).
- Card 2, `--parent` with `--reply`: reachable (`post --parent --reply 1 t`).
- Card 2, `--parent` with no orch strand recorded: reachable. The walk ends at the prime, and an empty orch state gives `OrchStrandOf` an unrecorded answer.
- Card 4, `askerOrchStrand` row from beam `b`: reachable in the test.
- Card 5, the marked-line size refusal (a body that fits the plain line but not the marked one): reachable through `Post`.
- Card 5, `Reply` answering a strand's `PostWait` with `OutcomeReply`: reachable live. The same answer is unguarded on the reconnect path (F1).
- Card 5, `Replied` set by a reply from the receiver a marked message was forwarded to: reachable. An unread marked message is forwarded, then read, then replied to.
- Card 6, `--expect-reply` with `--reply`: reachable.
- Card 7, `isChaseable` excluding a program holder: reachable. A marked post goes to a program that has read once.
- Card 7, `isChaseable` excluding an unread message: reachable.
- Card 7, the chase record leaving the `CREEL` mailbox unknown: reachable. But the same site recurs unguarded on the sender-notice record (F2).
- Card 7, `repairChases`: reachable through a seeded journal with no `chased` record.
- Card 7, sender-notice branches: all reachable through rows. The branches are a listed strand, a program in `readers`, EXTERNAL, a program that never read, and a strand no longer listed.
- Card 7, a store failure of the notice: raw I/O, exempt.
- Card 7, the hold-owner branch: reachable.
- Card 7, `settledOutcomeLocked` answering unanswered: reachable through a late `WaitReply`.
- Card 7, `carryRecords` "per program in `readers` whose read lies in a file past retention": the planned `readers` map is a set with no day, so the branch condition cannot be evaluated as planned (F3).
- Card 8, reply to a chase from a strand that does not hold the chased message: not reachable. That matches the card's own fail-loud guard claim:
  - `Forward` refuses a read message (`verbs.go:551`).
  - `releaseGoneHolder` moves only unread mail (`sweep.go:208-211`), and so does `Withdraw`.
  - The chase itself cannot leave its holder. Card 8 refuses a forward of it, and the sweep journals it undelivered, since its sender `CREEL` is no strand (`sweep.go:220`).
- Card 8, forward of a chase: reachable when the holder forwards the unread chase. The refusal has to sit before the `already read` case for a read chase, but the card's row uses an unread chase, so it is reached.
- Card 8, reply to a `CREEL` notice: reachable, because the notified sender holds the notice. The card's premise that the `CREEL` program holds no mailbox does not hold (F2).
- Card 9, the widened subjects and the holder-gone settle: reachable. A read marked message sits with a removed holder that holds nothing else; the Ack wake reaches `sweepWake` (`deliver.go:84-87`).
- Card 10, `--wait-reply` unanswered exiting 1: reachable. A strand may post to itself, since `checkReceiverLocked` has no self check (`pool.go:199-205`).
- Card 11, refusal from a pair: reachable through `resolvePrime`.
- Card 13, `"peer-messaging"` refused as unknown: reachable.
- Card 15, `ErrNoRunningTurn` and `refuseInterruptKey` wordings: reached by tests. `Runner.Inject` has no production caller, which the card already bounds.
- Card 15, the row asserting `Command()` has no `send` or `interrupt` child: reachable.
- Card 16, scan fixture rows and tree rows: reachable. The tree passes after cards 3 and 12, because `hold.go` holds the only Go literal naming the tool.

**Findings**

F1:
- severity: MEDIUM
- class: design
- location: card 5, `Store.Reply` together with the unedited `Store.settledOutcomeLocked`
- summary: Card 5 answers a strand's or program's reply wait only at the moment `Reply` runs. A `post --wait-reply` whose connection drops resumes with `VerbWaitReply` (`client.go:180-181`). `settledOutcomeLocked` finds a standing reply only for an EXTERNAL sender (`pool.go:378-383`: kind reply, undelivered, external holder). A reply that lands during the gap is therefore never handed to the resumed wait. Card 7 then sees the message as replied and chases nothing, so the wait hangs until `--timeout`, or forever without one.
- fix: card 5 also makes `settledOutcomeLocked` answer a reply wait with the newest reply whose `ReplyTo` is the id, for a non-EXTERNAL sender, plus a row covering it.
- evidence: `internal/creelengine/pool.go:375-387`, `internal/creelengine/client.go:180-181`, card 5's `Reply` bullet.

F2:
- severity: MEDIUM
- class: design
- location: card 7, `pool.applyPosted`, and card 8's `CREEL`-notice refusal
- summary: Card 7 keeps the `CREEL` mailbox unknown only for a `posted` record that carries `Chased`. The sender notice of an `unanswered` outcome is a `posted` record from `<shortname>:CREEL` without `Chased`. `applyPosted` marks every program sender's mailbox known (`fold.go:303-305`), and `checkReceiverLocked` accepts any known program as a receiver (`pool.go:203`). So after the first notice, `lym creel post <shortname>:CREEL …` is accepted and the post sits unread forever. Card 8's refusal rationale ("the `CREEL` program holds no mailbox") is then false. The guarded site recurs on the notice path, and that path is unguarded.
- fix: card 7 exempts every `posted` record whose sender is `CREEL` from the `programs` mark, not only chase records, and adds a fold row for a notice.
- evidence: `internal/creelengine/fold.go:303-305`, `internal/creelengine/pool.go:198-205`, card 7's Journal-and-fold bullet, card 8's Intent paragraph 4.

F3:
- severity: LOW
- class: consistency
- location: card 7, `pool.carryRecords` and the `readers` map
- summary: The carry branch fires per program in `readers` "whose read lies in a file past retention". The card plans `readers` as a map of the programs that have read once, with no record of which day held the read. As planned, that condition cannot be decided: the carry either repeats every retention or never fires.
- fix: plan `readers` as program → origin day, moved by `moveOrigins` the way message origins are.
- evidence: card 7, the bullets on `applyRead` and on retention; `internal/creelengine/fold.go:451-468` (`moveOrigins` keeps per-entry origin days).

Tool calls:
1. Read `pool.go` `checkReceiverLocked` and the `VerbWaitReply` reconnect site in `client.go`. My context had neither the receiver rule for programs that have posted nor the client's resume verb.
