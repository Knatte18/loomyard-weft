# Fixer report — plan, round 1

Every finding's premise was checked against the cards and the code: `pattern/PATTERN-test-speed.md`'s "Untagged tests spawn nothing", `internal/fabricengine/worktreelist.go` (`gitexec.Run` of `git worktree list`), `internal/preflight/predicates.go` (`ResolveMode` calls `lyxcwd.Resolve`), `pattern/PATTERN-daemon-wakeups.md`'s backoff rule and card 8's `peer_other.go`.
All nine hold.

## Changes

- **F1** (card 13): `testmain_test.go` stays untagged with only `gitkit.HermeticGitEnv()` and `tmuxkit.Main`, so every tier has its `TestMain`; `delivery_test.go` carries `//go:build integration`, so `TestHubDelivery` runs in tier 2.
- **F2** (card 14): `cli_test.go` carries `//go:build integration`, so `TestCreelCLI` runs in tier 2 over a `hubforge` hub; the plain-repository row uses a `hubforge` or `gitkit` fixture inside that file.
- **F3** (cards 9 and 0): the sweep backs off from a 30-second base to a named 4-minute ceiling while two consecutive successful sweeps see the same strand-name set, returning to the base on a set change, a new post, a new inject or a failed sweep; the card states the departure from D15's fixed 30 seconds and why.
  The What-counts bullet carries the base and ceiling, the overview's constants list says "sweep backoff (30 seconds to a 4-minute ceiling)", and `TestSweep_RemovedHolders` gains the 30 s, 60 s, … ceiling, back-to-30 s row.
- **F4** (cards 7, 9, 10): `Store` owns the waiter registry (message id, inject id or holder, answered by the settling verb) and a per-receiver wake notification signalled on post, forward, inject and release.
  `Post` and `Reply` no longer name cards 9 and 10; `TestStore_Mail` gains a reply to an EXTERNAL message handed to an in-process registered waiter.
  Card 9's `Scheduler` subscribes to the wake notification and its sweep answers waiters through the registry; card 10's protocol binds a waiting connection to that registry.
- **F5** (cards 6, 7, 9): card 6 adds the `withheld` event to its list, its fold state and a `TestJournal_Fold` row.
  Card 9 states that an orch gets no worker and no `Idle` probe, and that one `withheld` event naming `creel-notices` is journaled per change of its unread set by the verb or sweep making the change; the orch row of `TestScheduler_Reminders` asserts one `withheld` line per new message and zero `Idle` calls.
  Card 7's `Log` reads the event, and its log row asserts a post to an orch journals one `withheld` line.
  Fixed slightly differently: the fix key puts the statement in card 9 only, but post and forward are card 7's verbs, and card 7's log row reading the event would otherwise depend on a later card (F4's rule).
  So card 7's `Post` and `Forward` write the event themselves, and card 9's sweep writes it for a return.
  `Inbox` is unchanged: it states the withholding for any orch caller and needs no event.
- **F6** (card 10): `TestDaemon_Lifecycle` gains the row where an EXTERNAL `post --wait-reply` loses its connection, a strand replies while no waiter is connected (journaled `undelivered`), and the reconnected waiter gets the reply at once with a `delivered` event journaled.
- **F7** (card 7): a Bound line states what a privileged inject lets through (any single-line text without a control character, slash commands included, into any non-orch strand's pane at its next idle), the TTL bound and card 9's pause, hold and orch holds.
- **F8** (cards 10 and 14): every as-a-strand row also sets the claim's `LYX_STRAND_NAME` to that strand's name; card 10's disagreeing-`LYX_STRAND_NAME` refusal row is skipped off Linux with a reason naming D4's Linux-only ancestry check.
  Card 14 has no such refusal row, so it needed no skip.
- **F9** (card 11): the background now attributes the import half to `TestCreelStore_DeliverySeamImports` and the no-tmux-command half, like the journal-writer half, to review.

## Disputed

None.

## Deferred

None.
