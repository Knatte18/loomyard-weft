---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/creelengine/server_test.go:704
    summary: waiting-post behaviours have no row - receiver_live on the pending and final responses, and no warning progress for a live receiver
    fix: In server_test.go, extend the row PostAnswersWhetherItsReceiverIsLiveAndAWaitingPostHearsTheWarningAtOnce, or add a row beside it, with a waiting post to the live strand alpha that is answered by a reply. Assert that no progress note carries a Warning, that the pending response has ReceiverLive true, and that the final response has ReceiverLive true. Also assert ReceiverLive false on the pending response of the dormant waiting post already in the row.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/creelcli/cli.go:444
    summary: no CLI test covers post --wait-reply to a not-live receiver, so a dropped or doubled warning line or a lost receiver_live goes unnoticed
    fix: In internal/creelcli/cli_test.go, add a --wait-reply case to the new post receiver_live subtest (or a second subtest) against the dormant strand beta with a short timeout. Assert that stderr holds the warning line exactly once. Also assert in that subtest that a post to a program receiver has no receiver_live key, if the rig makes one cheap.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/creelengine/client.go:213
    summary: a waiting post resumed after a reconnect is sent as wait-reply with an empty postStamp, so its final response and the CLI envelope lose receiver_live
    fix: In the post branch of cli.go, keep ReceiverLive from the first pending response (read it in the progress callback or through a pending hook) and fall back to it when the final response carries none. Alternatively state in the doc.go of creelcli that the field is absent from a post resumed after a dropped connection.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/creelengine/client.go:163
    summary: the CallWatching doc says it hands over the progress responses of a waiting inject, but a waiting post now sends one and the CLI relies on it
    fix: In the doc comment of CallWatching, change "every progress response of a waiting inject" to "every progress response of a waiting inject or post".
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/creelcli/doc.go:43
    summary: the doc says the answer line follows any stall notes, but does not say that the new warning line comes before it
    fix: In creelcli/doc.go, add to the new post lines that the warning line is printed before the answer line.
  - id: F6
    severity: NIT
    class: consistency
    origin: handler
    location: internal/creelcli/cli.go:395
    summary: the post help text names interrupted in the answer but not receiver_live or the warning line
    fix: "In the Long help of the post command, add one short sentence that the answer carries receiver_live and that a receiver that is not live gets a `creel: warning:` line on stderr naming how to start it; keep the help under its size cap."
---

### [LOW] Waiting-post liveness behaviours have no test row

**Location:** internal/creelengine/server_test.go:704
**Issue:** The only waiting post in the protocol row targets the dormant strand and ends by client timeout. `ReceiverLive` on the pending response, `ReceiverLive` on the final outcome response, and the absence of a warning for a live receiver are never asserted. Breaking any of the three leaves the suite green.

### [LOW] No CLI test for a waiting post to a not-live receiver

**Location:** internal/creelcli/cli.go:444
**Issue:** The `warn` callback, the `CallWatching` wiring and the trailing `warn(resp)` are only run by plain posts in the new CLI test. A waiting post could print the warning twice or not at all, or lose `receiver_live`, without a failing test. The omit-when-unknown branch (`ReceiverLive == nil`) is also unpinned at this layer.

### [LOW] receiver_live is lost when a waiting post reconnects

**Location:** internal/creelengine/client.go:213
**Issue:** After a dropped connection the client resumes a waiting post as `VerbWaitReply`, which the daemon answers with an empty `postStamp`. The final response then has no `ReceiverLive`, and the envelope of a post that did report it at first omits the field. The warning is rightly not replayed. The case is rare and the message still queues, hence LOW.

### [LOW] CallWatching doc names only the inject

**Location:** internal/creelengine/client.go:163
**Issue:** The doc says progress responses of "a waiting inject" reach the callback. A waiting post now sends one, and the post command depends on it.

### [NIT] creelcli doc does not place the warning line

**Location:** internal/creelcli/doc.go:43
**Issue:** The line about the answer line on stderr says it follows stall notes. The new warning line is printed before the answer line and the doc does not say so.

### [NIT] Post help omits the new output

**Location:** internal/creelcli/cli.go:395
**Issue:** The help documents `interrupted` as an answer field but says nothing of `receiver_live` or the warning, so an agent reading the help does not learn of them.

## Rejected

- Lens code-correctness F1 (the `Reading` outcome path at `server.go:461` carries no `receiver_live`): that response is built for a read-waiter's messages, not for the outcome of a post, and it never carried `Interrupted` either. The fork itself could not show the path is reachable for a post.
- Lens code-affected-set note on `PATTERN-refusal-way-forward`: the entry covers webster, shed and loom refusals only, so the warning is outside it; the fork raised no finding on it.
- Stale liveness read before `s.do` (code-correctness): the value is the same listing the receiver check reads, the message queues either way, and the answer is advisory.
- The fixture edge of an empty `Worktree` in the dormant-strand warning: an unreachable state for a listed strand.
