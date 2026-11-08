---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/slackengine/relay.go:693
    summary: no test pins the terminal-notice guard that drops awayFrom, the turned-and-no-error condition, or the set's clearing on an early return
    fix: In internal/slackengine/relay_test.go TestRunRelay add rows beside the row at line 991. (a) An operator message turns the presence to slack and the presence then goes to terminal; assert every orch gets the terminal notice. (b) A forward that fails after turning the presence; assert the orch still gets the slack notice. (c) A set taken by a wake that sends no notice does not suppress a later notice.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/slackengine/inbound.go:648
    summary: the orch is marked as woken when forward posted nothing (empty body, oversize message, give-up after the retry window), so its presence notice is skipped though nothing woke it
    fix: Have forward report whether it posted to the orch, for example a second bool result or a distinct return from the three early-handled paths, and set in.awayFrom[orch] only when it did. Add a handle row with an oversize or empty message that turns the presence and assert the orch still gets the notice.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/slackengine/inbound.go:646
    summary: when the first forward fails after turning the presence, the retry sees the presence already slack, turned is false, and the orch is never marked, so the notice wakes it a second time
    fix: Record the orch in a pending set when awayOnOperatorMessage turns the presence, before forward runs, and move it to awayFrom only when a post succeeds, so a retry of the same message still counts. Keep the pending set under in.mu like awayFrom. Add a handle row where the first Post fails and the retry succeeds, and assert the orch gets no notice.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/spec.go:152
    summary: the TurnHookCommand doc says the hook runs at turn ends that hand the session no creel mail, which the diff makes false
    fix: Reword spec.go line 152 to say the command also runs at each turn end of the session's main thread, whether or not creel mail is handed to the session.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/slackengine/backoff.go:74
    summary: the reset of the wait to the floor after a long poll is not pinned, nor the boundary of elapsed equal to the floor or a floor raised by a rate limit
    fix: In outbound_test.go TestOutboundRead add a row where the fake clock advances nothing for two reads (waits double), then Ceiling for one read (wait none), then nothing again; assert the next wait is twice the floor. Add a row with the floor raised by a rate limit and a poll of exactly the floor's length expecting none.
---

### [MEDIUM] Untested guards in the presence-notice skip

**Location:** internal/slackengine/relay.go:693
**Issue:** The only new relay row covers the slack case with a successful forward. Removing the `presence != PresenceSlack` reset, the `err == nil` condition or the clearing of the set before the early returns leaves every test green, and each is a branch the diff adds.

### [LOW] Orch marked as woken when forward posted nothing

**Location:** internal/slackengine/inbound.go:648
**Issue:** `forward` returns `(true, nil)` without a Post for an empty body, an oversize message and a give-up after `retryWindow`. `handle` marks the orch in each, so the notice that would tell it the presence is now slack is dropped though no mail reached it. The doc comment says "the forward stood", which the condition does not test.

### [LOW] Retried forward loses the skip

**Location:** internal/slackengine/inbound.go:646
**Issue:** The first attempt writes the presence and fails the Post; the retry finds the presence already slack, so `turned` is false. The orch gets the mail and the notice, the double wake this change removes, on the failure path.

### [LOW] Stale TurnHookCommand doc

**Location:** internal/shuttleengine/spec.go:152
**Issue:** The field doc limits the Stop run to turn ends that hand out no creel mail. The merged Stop command now runs the hook first (`settings.go:134-139`).

### [LOW] Backoff reset and boundary unpinned

**Location:** internal/slackengine/backoff.go:74
**Issue:** The new row advances the clock by `Ceiling` every read, so `w.current = w.floor.get()` and the `elapsed >= floor` boundary are not exercised.

## Rejected

- Lens code-tests F-C (NIT, test economy: two new `TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock` rows overlap `TestTurnHook_MergedCreelStopRunsItOnEveryTurnEnd`): the rows add the payload-on-stdin and discarded-output assertions the other test lacks, so they are not a duplicate.
- Focus point on `in.awayFrom[orch] = true` made without `in.mu`: `handle` is reached only from `drain` and `retryOwed`, both of which hold `in.mu`, and both run on the wake loop's goroutine.
- Focus point on a notice lost for an orch skipped by `takeAwayFrom` before the early returns: the early returns mean no notice was due, and a change back to terminal clears the skip set, so no notice an orch should get is lost beyond F2 and F3.
- Focus point on `elapsed >= floor` letting the loop run hot: a poll that counts waited at least the floor, so reads start at most once per floor, and a shorter poll backs off through `afterIdle`.
- Doc comments and the merged Stop command's order: the `doc.go` and `settings.go` comments match the code the diff leaves; the one stale text is F4.
- Scope against the board entry: all three ideas land (turn hook on every Stop, the late relay read, the skipped presence notice), and the diff changes nothing the entry does not license.
