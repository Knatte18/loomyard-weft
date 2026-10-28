# Decision record — compact-inject-draft

## Goal

A cycle command creel injects into an idle orch is journaled `injected` when the session took it, and mail to an idle strand always wakes it: the idle waiter or the bell for a strand whose session runs creel's hooks, the notice or the reminder for any other.

## Scope

- Issue #897: shuttle's paste confirmation for a send that lands on a session start (`/compact`, `/clear`) stops failing a send already seen submitted when later text stands in the input box.
- Issue #899, added by the parent: creel's delivery drops the third class of strand, one with neither creel's hooks nor the pane path, so a strand whose session lacks `CreelHooks` keeps the notice and the reminder whatever its provider.
- The docs of `internal/shuttleengine`, `internal/creelengine` and `internal/creelcli` follow.

Out of scope: the orch watcher's handling of an `unconfirmed` cycle step, which already waits for the compaction boundary; retyping a command inject creel journaled unconfirmed.

## Decisions

### D1 — The incident's cause, from the orch's records

Decision: the change fixes the cause the transcript and the creel journal show, not the one the board entry suspected.
The orch's Claude transcript records `/compact` submitted at 13:44:06.083, 150 ms after creel's `sending` record for #107, and its compaction boundary at 13:44:43.114.
The orch run's `settings.json` dates from 2026-10-27 and carries no `hook-wait` entry, so no `asyncRewake` waiter ran.
The confirmation failed at 13:44:25, inside the 30 s submit window, when the box held the operator's text typed during the compaction, which `confirmPasted` reads as `ErrForeignBoxText` and creel maps to `ErrNotLanded`.
The `draft` stall at 13:44:43 is that same operator text, submitted at 13:44:44, and the cycle's color, plugins and resume injects followed at 13:45:15–19.

Rationale: the records place the submission and the failure exactly; the suspected hook race cannot have run in a session without the hook.

### D2 — A submitted landing send never fails on later box text

Decision: in `confirmPasted`, a send with a `landingStart` that reads its input box empty after the paste counts as submitted.
From then on a box holding any text, or no readable box, is read again and sends no key, and a window or read cap that runs out returns nil, as an empty box does today.
A send that has not read its box empty keeps today's rules: its exact text gets the guarded Enter, the text beside other text fails `ErrSentTextInDraft` and other text fails `ErrForeignBoxText`.

Rationale: an empty box after the paste is the evidence every other paste already lands on; the landing send waits past it only for the session start, and text arriving after it is a new draft, never this send's.
Bound: it lets through only a landing send whose box read empty after its paste; it never sends a key after that read, so an operator's draft is never submitted or cleared.

### D3 — Every strand without creel's hooks keeps the pane path

Decision: creel's `Strand` loses its `PaneMail` field, and the delivery probe rings or bells a strand marked `CreelHooks` and types the notice or the reminder into every other strand.
`internal/creelcli` drops `hasHookDelivery` and its provider lookup.

Rationale: a Claude session launched before `creel-hook-post`'s deploy, by `lym shuttle open` or by `lym reed add` has a hook-delivering provider but no idle waiter, so it fell into neither branch and its mail sat unread until a tool call or a hand read; the parent asked that mail to an idle strand always wake it.
Bound: such a strand may also claim at a tool boundary through its older hook; the notice is reserved before it is typed and a claim reserves before it hands out, so each message is still delivered once, and agent mail stays reminder-only.

### D4 — Parent decision: issue #899 folded into this run

Decision: the scope covers issue #899 as well as #897, and the change description closes both, as the parent `ly:orch` decided by message on 2026-10-28.

Rationale: both are creel delivery into an idle orch strand, and the operator chose one run over two.

### Added after Discussion (parent, 2026-10-28): Scope covers issue #899 as well as #897

Decision: The task fixes issue #899 (https://github.com/Knatte18/loomyard/issues/899) beside #897: mail to an idle Claude strand without the CreelHooks reed field must wake it through the pane path, as mail to every idle strand must. The change description closes both issues. This confirms D4.

Rationale: Same code as #897, creel delivery into an idle orch strand; the operator chose to fold it into this run rather than start another.

## Constraints

- `PATTERN-pane-text-injection`: typing stays inside shuttle's guarded send and creel's `Delivery` seam.
- `PATTERN-shuttle-provider-seam`: the change in shuttle stays provider-neutral; nothing Claude-specific leaves `claudeengine`.
- `PATTERN-test-speed`: confirmation and delivery tests run on the injected clocks and fakes they use today.

## Auto-mode assumptions

- Claude Code's input box stays readable during a compaction, since the operator typed into it then; a landing send whose box reads unreadable after the paste keeps today's rules.
- Strands with no provider tag already took the pane path, and D3 keeps that.

## Open risks

- A landing send whose box shows foreign text on its very first read, before any empty read, still fails `ErrForeignBoxText`.
- A Claude strand without `CreelHooks` gets a typed notice for program and operator mail again, which a session with the older claim hook may also have claimed; the reservation keeps it to one delivery.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - creel-help-old-pane-mail-rule (class consistency, severity MEDIUM, open in rounds 1)
  - interrupt-refusal-names-removed-distinction (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1, D2: a compact send whose box reads empty and then holds other text, with no session start inside the window, succeeds and sends no further key.
- D2: a compact send whose box reads empty and then has its compaction session start succeeds, as today.
- D2: a compact send whose box holds other text before any empty read still fails `ErrForeignBoxText`.
- D3, D4: an idle strand without `CreelHooks` gets its notice and its reminder, and a `CreelHooks` strand gets neither and is rung or belled.
- D3: creel's hub delivery lists a Claude strand without `CreelHooks` for the pane path, and one with it for the wake.
