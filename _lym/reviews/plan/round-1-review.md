---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-receive-and-confirm.md:94
    summary: Card 9's reply guard refuses a delivery with an unknown reply-to before the confirm, so the reply-to case of card 7's drop row fails at card 9's gate, and card 9 does not list that test
    fix: >-
      On card 9, add `plan:internal/creelengine_test#TestDaemon_CrossHubDeliver` under `**Edit:**`.
      In its Intent, change the reply-to case of the row "A `deliver` of a real pending id differing in each of sender, receiver, reply-to, mark and post time is refused as a drop":
      its `ReplyTo` names a real `outbound` entry of the receiving hub, sent to the sending hub by the delivery's receiver, so the delivery passes the reply guard and the confirm answers no.
      The overlay accepts this target.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-outbound-worker.md:64
    summary: Card 8 maps only plain refusals to quoted text, so a wrong hub path or a coded refusal from the socket reaches the CREEL notice with no code, no quoting and no test row
    fix: >-
      On card 8, state that `Client.exchange` returns a coded refusal as `daemonRefusal` too, and that every other `Client.exchange` failure, the hub-path mismatch and undecodable JSON included, takes a code creel writes, such as `bad-answer`, with no socket text in the reason.
      Add a `TestScheduler_Outbound` row in which a fake socket answers with a foreign `HubPath` holding a quote and a line break, and another with a set `Code`, each asserting the notice text.
      On card 7, let an `answer` set its own `HubPath`, so `listenHubFake` writes the rig's hub path only when the answer leaves it empty.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/09-replies-across-hubs.md:27
    summary: Card 9 changes what ComputeReplies counts but lists internal/reportengine/doc.go only under Uses, so its Replies section goes false
    fix: >-
      On card 9, move `internal/reportengine/doc.go` from `**Uses:**` to `**Edit:**`, and state in the Intent that its Replies section and the `RoleReplies` field comments name the `inbound` and `outbound` counts.
      The overlay accepts this move.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/01-hub-registry.md:56
    summary: The way forward says any lym creel verb records the hub again, yet lym creel hubs never does, and a verb within a minute of the last record does not either
    fix: >-
      On card 1, make the corrupt-record way forward name `lym creel inbox` instead of any verb.
      On card 4, have `recordHub` skip the interval when the last record failed, and have `Record`'s caller note in the Intent that a removed file is recorded again by a verb past `hubRecordInterval`.
  - id: F5
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/05-hubs-verb.md:33
    summary: Card 5 adds hubs to the package-tree verb list of docs/overview.md and misses the verb list in that file's creel bullet
    fix: >-
      On card 5, extend the sentence on `docs/overview.md` so it also names the **creel** module bullet's list of `lym creel` verbs.
      The target is already on the card.
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/09-replies-across-hubs.md:40
    summary: Card 9's Intent has storeInboundLocked keep kind reply, which card 7 never states, and card 9 does not list that member
    fix: >-
      On card 9, add `plan:internal/creelengine#Store.storeInboundLocked` under `**Edit:**`.
      The overlay accepts this target.
---

Every proposed target change on card 9 was run together through `lym loom validate-plan --overlay`, and it reports no blocking finding, as the `{}` baseline does.

### [BLOCKING] Card 9's reply guard breaks a card 7 test row

**Location:** `_lym/plan/07-receive-and-confirm.md:94`, against `_lym/plan/09-replies-across-hubs.md:37`

**Issue:** Card 7's rows post plain mail, so the pending message has `ReplyTo` 0, and the row's differing delivery carries a non-zero `ReplyTo`.
At card 7 that delivery reaches the confirm, which answers no, so the row sees a drop and a notice.
Card 9 then refuses any non-zero `ReplyTo` with no matching `outbound` entry alongside the receiver rules, before the confirm, with no drop.
The row's assertion fails at card 9's gate, and card 9's Edit list does not name `TestDaemon_CrossHubDeliver`.
As written, card 9 cannot pass its own gate, which is why this stands at BLOCKING above the lens's MEDIUM.

### [MEDIUM] Socket-written text can pass around the notice's quoting

**Location:** `_lym/plan/08-outbound-worker.md:64`

**Issue:** D2 bounds a return notice to 200 escaped bytes of socket-written text, because `<shortname>:CREEL` mail is followed as the operator's own.
Card 8 turns only the `errors.New(resp.Error)` branch into `daemonRefusal`.
`internal/creelengine/client.go:244` puts the unbounded `resp.HubPath` into a plain error, and `client.go:248` returns `resp.Error` as a `codedRefusal` when the socket sets a code.
Neither is `daemonRefusal`, `connLostError` or `ErrWaitTimeout`, so the card's mapping gives them no code.
The hub-path branch is the first one a process binding the socket hits.
The sentence "whatever the socket that wrote it" states the intent, but nothing carries it, and `listenHubFake` always writes the rig's hub path, so no row can reach that branch.

### [MEDIUM] The reportengine package doc goes false

**Location:** `_lym/plan/09-replies-across-hubs.md:27`

**Issue:** `internal/reportengine/doc.go:206` defines Expected as the `posted` entries and Replied as the `posted` reply entries.
Card 9 counts `inbound` as `posted` and counts `outbound` entries with `Answers` as `Replied`, and it keeps the doc under `**Uses:**`.
Both the affected-set and repo-invariants lenses reported this.

### [LOW] The record's way forward is not true of every verb

**Location:** `_lym/plan/01-hub-registry.md:56`

**Issue:** Card 1's corrupt-record way forward says any `lym creel` verb records the hub again.
Card 5's `hubs` verb starts no daemon and writes no record, and card 4's interval skips a record within a minute of the last one.
A corrupt record fails every record attempt, so an operator who removes the file may run a verb inside the minute and see no entry.
The none refusal's wording comes from D1 and stays; only the card's own corrupt-record text and the interval change.

### [LOW] A second verb list misses `hubs`

**Location:** `_lym/plan/05-hubs-verb.md:33`

**Issue:** `docs/overview.md:323` and the creel bullet at `docs/overview.md:512` both list creel's verbs.
Card 5 names only the first, and card 10's edit of the bullet covers the daemon and the trust sentence.

### [LOW] Card 9 changes a member it does not list

**Location:** `_lym/plan/09-replies-across-hubs.md:40`

**Issue:** Card 7's `storeInboundLocked` says nothing of the record's kind, and card 9 says it "keeps kind reply", so the member's work lands in card 9.
Card 9's Edit list names `Store.Deliver` and not `Store.storeInboundLocked`.

## Focus verdicts

- Fidelity: every step of D1 to D9, the added drop rule and the Constraints is carried; D2's return and start cap by cards 6 and 8, D3 steps 3 to 6 and the notice window by card 7, and D4 by cards 3 and 9.
- Card order: each card's `Uses` sits on earlier cards, and the daemon-wakeups entry lands with `outbound.go` in card 8; F1 is a later card breaking an earlier test, not an order fault.
- Attack surface: card 7 states what `deliver` and `confirm` skip and what bounds them, card 2 states the Guard classification, and card 5 and card 6 state theirs.
- Reachability: every named refusal has a named input and test, and no upstream check rejects it first, with F2 the one gap.
- Test economy: no planned test duplicates an existing one, and card 10 plans no test of prose.
- Edit glyphs: the `.go` Edit targets are members, and no card changes a signature that needs the arrow; F6 is the one unlisted member.
- Research and board: no card leaves a fact to find out, and card 10 files a note with a valid `research` label and leaves this task's entry alone.
- Verify lines and PATTERN fit: no card has a `**Verify:**` line, and the six named entries hold.

## Rejected

- The way forward in card 1's none refusal: the wording is D1's, and the answer key is never the subject of a finding; only the card's corrupt-record text is kept in F4.
- `Extends` on `docs/overview.md` and the pattern backgrounds in cards 2 and 5: each adds an item to a list and changes nothing already there.
- Hand-built hub directories in the tier0 tests of cards 1 and 4: `internal/hubgeom/hubgeom_test.go` sets the same tier0 precedent, and `hubforge` is barred from tier0.
- Mail swept undelivered when another hub posts to an orch with no live strand: D4 prescribes it and its bound states it.
- `cmd/lym/sandbox_coverage_test.go`, the affected-set lens's unverified lead: it counts modules by CLI, `creel` is already one, and coverage is module-level, so `hubs` forces no entry.
- Card 6's claim that no test asserts the old refusal: the repo-invariants lens left it unconfirmed, and the affected-set lens confirmed it holds.
