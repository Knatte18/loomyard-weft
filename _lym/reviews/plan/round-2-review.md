---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/plan/05-hubs-verb.md:11
    summary: Cards 5 and 2 list docs/overview.md, internal/creelcli/doc.go and PATTERN-config-layers.md under Extends while their Intent rewrites existing list lines in them
    fix: "In 05-hubs-verb.md move `internal/creelcli/doc.go` and `docs/overview.md` from the **Extends:** group to the **Edit:** group (after `internal/creelcli#TestCreelCLI`) and delete the then-empty **Extends:** group. In 02-creel-config.md move `pattern/PATTERN-config-layers.md` and `docs/overview.md` from **Extends:** to **Edit:** (after `cmd/lym#configStrictnessDegradingSet`) and delete the empty **Extends:** group. Both cards already carry an ImpactSummary. Checked with lym loom validate-plan --overlay: the combined overlay reports no blocking finding beyond the baseline."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/plan/01-hub-registry.md:51
    summary: The none-refusal way forward says to run any lym creel verb, which is false of lym creel hubs, the verb card 5 adds
    fix: "In 01-hub-registry.md change the quoted none refusal to `creel: no hub with the shortname \"<s>\" is recorded on this machine; way forward: run lym creel inbox in that hub once`, dropping the words `any lym creel verb, such as`. D1's acceptance criterion needs only the shortname and `lym creel inbox` named. Keep `TestLookup`'s row asserting that `lym creel inbox` is named."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lym/plan/08-outbound-worker.md:52
    summary: Card 8 maps most Client.exchange failures to a creel code but leaves the neither-word answer without a reason, and names no test input for undecodable JSON or for a hostile refused Reason
    fix: "In 08-outbound-worker.md (1) add to the code list after `bad-answer` is introduced that an answer with neither `accepted` nor `refused` also gives `bad-answer` with none of its text in the reason; (2) in the row `An answer of neither word and an unknown-verb refusal are retried, not returned at once` add that at the hour the notice gives `bad-answer` for the neither-word answer and the quoted text for the unknown-verb refusal; (3) name the input of the row `A return notice quotes an answer of over 200 bytes...` as a hubFake answering Delivery refused with a Reason holding a quote, a backslash, a line break and over 200 bytes, so the quoting of Response.Reason is reached; (4) add a row for undecodable JSON: a hubFake writing a non-JSON line is retried and at the hour the notice gives `bad-answer` with none of that line. In 07-receive-and-confirm.md let `listenHubFake`'s answer return either a Response or a raw line, written verbatim, so row (4) can reach the decoder. These are text changes only; no target is added, dropped or moved."
  - id: F4
    severity: LOW
    class: scope
    location: _lym/plan/05-hubs-verb.md:27
    summary: The hubs verb states no behavior when seam.registryDir fails, though card 4 words that cause for the daemon
    fix: "In 05-hubs-verb.md, in the paragraph that lists the verb's refusals, add that a failing `seam.registryDir` is propagated raw, its cause naming `XDG_STATE_HOME` and `HOME` (standalonestate's error text), and add a `TestCreelCLI` row with `seam.registryDir` returning an error that is refused naming that cause. Text change only; no target changes."
---

### [MEDIUM] Extends used where the card rewrites existing lines

**Location:** `_lym/plan/05-hubs-verb.md:11`, `_lym/plan/02-creel-config.md:20`

**Issue:** Card 5 lists `internal/creelcli/doc.go` and `docs/overview.md` under **Extends:**, and its Intent says `internal/creelcli/doc.go`'s verb list, `docs/overview.md`'s `internal/creelcli/` line and the verb list in the **creel** module bullet "name `hubs`".
Those are existing lines (`internal/creelcli/doc.go:1`, `docs/overview.md:323` and `:512`) that gain a word, so the card changes content already there.
Card 2 does the same: its Intent says `PATTERN-config-layers`' degrading-caller list gains `creelconfig`, and that `docs/overview.md` "names it in the creel bullet", yet both refs sit under **Extends:**.
The rubric reserves `Extends` for adding to a ref without changing anything already there.
This round's focus asks that each `docs/overview.md` list of `lym creel` verbs is covered by an edit; card 5's prose covers both lists, but under the wrong type.

### [MEDIUM] Way forward of the none refusal is false for `lym creel hubs`

**Location:** `_lym/plan/01-hub-registry.md:51`

**Issue:** The none refusal tells the reader to "run any lym creel verb, such as lym creel inbox, in that hub once".
A hub is recorded only by its daemon, at start and at a served verb (card 4's `recordHub` in `daemon.serve`).
Card 5's `lym creel hubs` reads the record and dials the socket, "starts no daemon" and "never writes the record", so running it in that hub records nothing.
The text is therefore false for a verb the same task adds.
The decision record's acceptance criterion needs only `lym creel inbox` named.
The corrupt-record way forward in the same card, card 4's `registryErr` text and card 5's corrupt-record refusal are true of every verb and path they name: removing the file by its path and then running `lym creel inbox` records the hub at once if the daemon's last record failed and within `hubRecordInterval` otherwise, and `XDG_STATE_HOME` or `HOME` is what `stateRootFor` reads.

### [MEDIUM] Card 8's failure mappings leave a neither-word answer and two test inputs unnamed

**Location:** `_lym/plan/08-outbound-worker.md:52`

**Issue:** The return notice is `CREEL` mail an agent follows as the operator's own, so each failure path needs a stated reason with no unquoted socket text and a test input that reaches it.
Against `internal/creelengine/client.go` the paths are: write and read failures (`connLostError`, mapped to `connect-failed`), a failed spawn (mapped), `ErrWaitTimeout` (`timeout`), a plain or coded `!OK` answer (`daemonRefusal`, quoted), a hub-path mismatch (`bad-answer`, row at line 99) and undecodable JSON (`bad-answer` by the catch-all).
Three gaps remain.
First, line 52 lists "an answer with neither word" as a transient answer, but it is not a `Client.exchange` failure, so the line-67 catch-all does not cover it and no code or quoting is stated for its reason.
Second, nothing names an input that reaches the undecodable-JSON path: `listenHubFake` (card 7, line 89) writes a `Response` only, so it cannot emit a non-JSON line.
Third, the row at line 98 quotes "an answer of over 200 bytes" without saying which answer; the hostile rows at lines 99 and 100 cover a hub-path mismatch and a coded `OK: false`, and the `refused` `Reason` is exercised only by a real receiver's id clash, so the quoting of `Response.Reason` has no hostile input.

### [LOW] `lym creel hubs` states no behavior for a failing registry directory

**Location:** `_lym/plan/05-hubs-verb.md:27`

**Issue:** Card 4 words the cause of an unreadable per-user state directory for the daemon, but card 5 reads `seam.registryDir()` directly and lists only the corrupt-record and invalid-`creel.yaml` refusals, plus "propagated raw" for other reads.
It is unstated that a failing `registryDir` is propagated and what it reads.
The raw text from `standalonestate` already names `XDG_STATE_HOME` and `HOME`, so the fix is one sentence and one row.

## Checked and clean

- Card 7's rows that send a non-zero `ReplyTo`: the reply-to row of the drop row and the reply-to rows of the held-id-clash row.
  Card 9's guard stands after step 3's duplicate check, so the clash rows still refuse as a clash.
  Card 9 edits `TestDaemon_CrossHubDeliver` and has the drop row's `ReplyTo` name an `outbound` entry with its `remote-delivered` record, written before `startReceiving`, sent to the sending hub by the delivery's receiver, so the row still reaches the confirm and drops.
  Card 8's rows send no `ReplyTo`.
- Card 9's `Edit` list covers what `Store.Deliver`, `Store.storeInboundLocked` and `ComputeReplies` change: `internal/reportengine/doc.go` Replies section, `RoleReplies`, `ComputeReplies`, `TestComputeReplies`, `internal/creelengine/doc.go`, `Store.Reply`, `Store.Forward`, `releaseGoneHolder`.
  No other doc or comment states the old counting; the `docs/overview.md` and `reportcli` texts on the replies report stay true.
- `PATTERN-daemon-wakeups`, `PATTERN-child-processes` and `PATTERN-cwd-resolution` are met by the cards as stated.
