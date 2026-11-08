---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/plan/06-two-token-login.md:73
    summary: card 6 fills the two-parameter newDialer seam with slackengine.NewDialer, which card 2 gives four parameters
    fix: "In card 6's Intent, replace the sentence 'wireProduction fills newDialer with slackengine.NewDialer and liveHubs with liveHubGeometries' by one saying wireProduction fills newDialer with a closure `func(baseURL, appToken string) slackengine.Dialer { return slackengine.NewDialer(baseURL, appToken, &http.Client{}, slackengine.HelloTimeout) }`, the client built as newSlack's is, and liveHubs with liveHubGeometries. No target line changes, so the fix needs no overlay run."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/plan/09-socket-holder.md:81
    summary: cards 9 and 10 append the move-the-hubs-record-aside way forward to every LiveHubs error, which contradicts card 6's promise that a geometry failure never sends the operator to move a sound record
    fix: "Three text edits, no target changes. (1) Card 6, the liveHubGeometries paragraph: say a failed hubregistry.Live read is returned wrapped as `slack: read the machine record of hubs <dir>/hubs.json: %w`, which keeps hubregistry's own message in the chain (an undecodable record already names the file and the way forward of removing it, after which `lym creel inbox` in each hub records it again), that a hub whose geometry fails is an error naming that hub's path and the geometry's own error with its own way forward, that both are propagated errors no row reaches, and that the socket holder's and the keeper's notices quote the seam's error as it is. Delete the clause 'so no notice sends the operator to move a sound record aside'. (2) Card 9, the paragraph 'While the LiveHubs read fails ...': replace 'naming the hubs record's path from the seam's error, with the way forward of moving that record aside, after which lym creel inbox in each hub records it again' by 'carrying the seam's error as it is, which names the path and its way forward'; in the test bullet on unmapped events, say the scripted LiveHubs error carries the record path and that the notice carries it. (3) Card 10, the paragraph on a map that cannot be read: replace 'or naming the hubs record, with moving it aside and `lym creel inbox` in each hub a minute on as the way forward' by 'or carrying the LiveHubs error as it is, which names the hubs record or the failing hub and its way forward', and make the scripted LiveHubs error of the 'LiveHubs failing' test row carry the record path."
  - id: F3
    severity: LOW
    class: scope
    location: _lym/plan/13-relay-push-loop.md:101
    summary: missingTokenExit skips the prime-channel and old-channel posts without a bot token, but no TestRunRelay row reaches that branch
    fix: "In card 13's TestRunRelay first row (HubWithNoTokenOrNoSetupIsRefusedAndTakesNoLock replacement), add a clause: a relay with an app-level token and no bot token mails one notice naming the bot token and `lym slack login`, makes no chat.postMessage call although the map holds the prime's channel and a setup.json holds channel_id C7, and exits. No target line changes."
---

### [MEDIUM] Card 6 fills the `newDialer` seam with a function of another shape

**Location:** `_lym/plan/06-two-token-login.md:60` and `:73`.
**Issue:** The card declares the `slackCLI` seam as `newDialer func(baseURL, appToken string) slackengine.Dialer` and says `wireProduction` fills it with `slackengine.NewDialer`.
Card 2 creates `NewDialer(baseURL, appToken string, client *http.Client, helloTimeout time.Duration) Dialer`, and card 13 calls it with all four arguments.
The assignment does not compile as the card words it.
The implementer must invent the client and the hello bound, and the card's own production value for the bound (`HelloTimeout`) is left unsaid.
The fix names both.

### [MEDIUM] Cards 9 and 10 give every `LiveHubs` failure the hubs-record way forward

**Location:** `_lym/plan/09-socket-holder.md:81` and `:104`, `_lym/plan/10-channel-keeper.md:65` and `:89`, against `_lym/plan/06-two-token-login.md:71-72`.
**Issue:** Card 6 makes `liveHubGeometries` return two kinds of error: a failed `hubregistry.Live` read that names `<dir>/hubs.json`, and a hub whose geometry fails, named by that hub's path "never the hubs record, so no notice sends the operator to move a sound record aside".
Both come through the one `LiveHubs` seam.
Card 9 states its notice for any failing `LiveHubs` read as "naming the hubs record's path from the seam's error, with the way forward of moving that record aside", and card 10 does the same.
A geometry failure therefore reaches a notice that tells the operator to move a sound `hubs.json` aside, which is the outcome card 6 says it prevents.
The decision record's D1 and D3 word the notice for a failed `hubregistry.Live` read only; the geometry failure is the plan's own addition, so the plan must give it a coherent way forward.
`hubregistry.refuseUnreadable` already names the file and exactly D1's way forward for an undecodable record, so the notices can quote the seam's error instead of composing the advice themselves.
The errors the fix prescribes are propagated, so no row has to reach them, and the scripted errors of the card 9 and card 10 rows only need to carry the path the notice is asserted to name.

### [LOW] The no-bot-token branch of `missingTokenExit` has no named row

**Location:** `_lym/plan/13-relay-push-loop.md:67` and `:101`.
**Issue:** The Intent says the exit posts the line to the prime's channel "with a bot token and a map holding the prime's channel" and to the old `channel_id` channel "with the bot token".
The test names a bot token alone (the post happens) and both tokens with no setup file.
No row has an app-level token and no bot token, so the guard that keeps the relay from calling Slack without a bot token has no input that reaches it.

## Rejected

- Card 5 against card 6 order: card 5 leaves `login` writing the bot token alone over the new record and states the gap until card 6; every `ReadToken`/`WriteToken` caller in the tree (`start.go`, `credentials.go`, `cli.go`, `files_test.go`, `inbound_test.go`, `relay_test.go`, `cli_test.go`, the beamcli integration test) is on card 5's edit list, and no card before card 5 names either function.
- Cross-references by card number in cards 4 to 13 (4→10, 5→6 and 13, 7→8, 9 and 13, 8→9, 9→13, 10→13, 11→12 and 13, 12→11) each match the card that carries the work.
- `doc.go` prescriptions of card 13: the sentences it drops (`WriteToken`/`ReadToken`, the inbound cursor, the bearer header, `PostMessage`, the inbound poll with its five-second floor, and the slackcli `login` and `setup` sentences) exist in the files today.
- Reachability of the outbound recovery branches (`is_archived`, `not_in_channel`, `channel_not_found`, `errNoOpenChannel`, `errChannelRefused`) and the inbound `answer` join: each has a named input and a named row, and no upstream check rejects it.
- `PATTERN.md` edit on card 6 with the declared line unchanged: the plan-checks spec asks the block on a card that edits `PATTERN.md`.
