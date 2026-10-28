# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates are merged and listed once under every lens that raised them.

- coverage F1 + guard-reach F1 — BLOCKING — fixed as proposed: card 7 adds a `pool.readers` map set only by a program's read (carried across retention), and the sender notice goes to a listed strand or a program in `readers`, so a program that only posted gets no notice; `TestJournal_Fold` gains the post-versus-read row.
- guard-reach F2 — BLOCKING — fixed differently: the proposal to route the prime-hosted tackler and arbiter's `--parent` by the run's pair is not made, because D1 resolves "from the caller's worktree" and gives the prime its own walk, and D3 names batten's run-notice walk, which batten's wiring at `internal/battencli/drive.go:448` already uses; the D1/D3 acceptance criterion covers agents held "in a pair and in a beam". The divergence for prime-hosted strands is recorded as Shared Decision `prime-hosted-asker-bound` so the parent can widen D1.
- way-forward F1 — BLOCKING — fixed differently: making `Forward` move a read, chased message, or seeking a parent decision, is not done; card 8's refusal names only the reply route, which answers the chased message, and Shared Decision `chase-forward-way-forward` records why D10's forward clause is left out (`Store.Forward` refuses a read message, `internal/creelengine/verbs.go`).
- coverage F3 + guard-reach F3 — MEDIUM/LOW — disputed, premise false: the plan follows the plan-writer stencil's rule that a branch upstream validation rules out is written as a fail-loud guard taking no reaching test, and both lenses confirm the upstream check (`Store.Forward` refuses a read message, `internal/creelengine/verbs.go`); the refusal itself is carried by card 8, so no decided step is dropped.
- coverage F2 — MEDIUM — fixed differently: card 4 adds `askerOrchStrand` and a `TestParentOrchStrand` row from the beam with its orch live, the case where it differs from `parentOrchStrand`; the three one-line `SetNotifier` closures are left to review rather than given a wiring test each.
- coverage F4 + row-validity F5 — MEDIUM — fixed as proposed: card 11 adds the `creelLog` seam and a non-parallel tier1 `TestRepliesVerb` with the pair refusal and the prime's `roles` envelope; it is a test of its own because the existing report-verb test runs in parallel.
- coverage F5 — MEDIUM — fixed as proposed: card 16 adds `goLiteralHits` and `testdataPromptRoots` with `FixtureRows` for a literal hit, a comment non-hit and a listed testdata directory.
- coverage F6 — MEDIUM — fixed differently: no unknown-subcommand run, since the shuttle group's pre-run may answer first in the test's directory; card 15 adds a `TestRunCLI_PositionalArgValidation` row asserting `Command()` registers no `send` or `interrupt` child.
- coverage F7 — MEDIUM — fixed as proposed: card 7 row, a `WaitReply` after the outcome is answered `OutcomeUnanswered` at once.
- coverage F8 — MEDIUM — fixed as proposed: card 7 row, a `chase-aged` record is journaled at each tick while the clock runs.
- coverage F9 — MEDIUM — fixed as proposed: card 7 states the batten row's read through `markNoticeRead` and adds a strand-sender row read through `Read` and `Ack`.
- coverage F10 — LOW — fixed as proposed: card 7 row, a reply after the outcome is still delivered.
- coverage F11 — LOW — fixed as proposed: card 7 row, a marked message held by a program is never chased.
- coverage F12 — LOW — fixed as proposed: card 7's EXTERNAL row asserts its outcome posts no notice.
- coverage F13 — LOW — disputed, every fix worse: an assertion on the stencils' static prose is itself a test-economy finding under the Plan-Review rubric ("so is a card that plans an assertion on a stencil's static prose"), and the stencil sites are scanned for `SendMessage` by card 16.
- coverage F14 — LOW — fixed as proposed: card 16's stencil surface reads every file under `contracts/stencils/` but its `.go` files, which the Go surface reads outside tests.
- row-validity F1 — MEDIUM — fixed as proposed: card 5 rewrites `Post_RefusesEmptyAndOversizedBody`'s overhead to the marked form and adds a body that fits only the plain line.
- row-validity F2 + forced-edits F1 — MEDIUM — fixed as proposed: card 7 narrows `internal/creelengine_test#isSweepWait` to `8s < d <= 50s`, and card 9 names it; `internal/creelcli#isSweepWait` is left, since no card-10 row drives a sweep.
- row-validity F3 — MEDIUM — fixed as proposed: card 7 creates `env.writeJournal` and `env.reopen`.
- row-validity F4 — MEDIUM — fixed differently: rather than classify by caller, card 7 and Shared Decision `chase-clock-cadence` put the chaser on a fixed one-minute tick whose deadline a wake keeps, so every chase wait is exactly `agedPersistInterval` and `isChaseWait` matches that one value.
- row-validity F5 — merged with coverage F4.
- row-validity F6 — LOW — fixed as proposed: card 13 changes the unknown-requirement row's input to `"peer-messaging"` and adds no row.
- row-validity F7 — LOW — fixed as proposed: card 1's `TestAsker` keeps the beam, prime and not-recorded rows and one pair row pinning the branch.
- row-validity F8 — LOW — fixed as proposed: card 6 adds a `read --envelope` assertion of `expect_reply`.
- row-validity F9 — LOW — fixed as proposed: card 2's refusal rows start the daemon after the refusal and read `creel log`.
- forced-edits F2 — LOW — fixed as proposed: card 5 moves `internal/creelengine/doc.go` under Edit and names the line form and Tool-boundary sentence it rewrites.
- forced-edits F3 — LOW — fixed as proposed: card 7 moves `internal/creelengine/doc.go` under Edit, naming the retention, Log and Delivery loop text.
- forced-edits F4 — LOW — fixed as proposed: card 7 lists `Scheduler.Stop` and its doc comment.
- forced-edits F5 — LOW — fixed as proposed: card 11 moves `internal/reportengine/doc.go` under Edit and rewords its opening.
- forced-edits F6 — LOW — fixed as proposed: card 2 rewords `postCmd`'s two existing form refusals to list the three forms.
- guard-reach F4 — LOW — fixed as proposed: card 7 drops the dead "is no chase itself" clause from `isChaseable`, stating why a chase is never chaseable.
- guard-reach F5 — LOW — fixed as proposed: card 8 refuses a reply to a `CREEL` notice that is no chase, naming `lym creel post <strand>`, with a reaching row.
- way-forward F2 — MEDIUM — fixed as proposed: card 7 lists `daemon.awaitOutcome`, which puts the message in the response for `OutcomeUnanswered`.
- way-forward F3 — MEDIUM — fixed as proposed: card 7's settle skips the notice to a sender creel no longer lists, logs a failed notice, answers the waits regardless, and a row removes the strand sender before the outcome.
- way-forward F4 — MEDIUM — fixed as proposed: card 12's parent directive says to follow a refused post's way forward, and the darn writer and Master asks point at it.
- way-forward F5 — LOW — fixed as proposed: card 7's sender notice names `lym creel log --id <id>` and asking again or deciding without the answer.
- way-forward F6 — LOW — fixed as proposed: card 5's `Reply` also answers a non-EXTERNAL sender's registered reply wait with the reply, as the creel doc's Mail section states, with a `TestStore_Mail` row.
