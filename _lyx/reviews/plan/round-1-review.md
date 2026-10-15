---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/17-creel-read-sets.md:26
    summary: The rewritten TestDaemon_Protocol read --all assertion duplicates the plain-read assertion just above it and no longer pins that the All field crosses the protocol
    fix: "In card 17's Intent, replace the sentence on TestDaemon_Protocol's ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread row (line 26) with one that makes its read --all assertion discriminate: before the --all call, a second client (for example programClaim(\"ly:BATTEN\")) and alpha itself each post one message to alpha, and the row asserts that a Request with All set hands out both messages, where plain read would hand out only the older sender's one. Keep TestDaemon_Protocol under Edit."
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/17-creel-read-sets.md:19
    summary: The Intent prescribes read --all with --from narrowing to one sender, but the rewritten Read_SetsAndInbox row drops the existing assertion that reaches that combination and names none in its place
    fix: "In card 17's Intent, extend the Read_SetsAndInbox sentence (line 25) so the row also reads with from set to batten and all set while b1 and an alpha message are still unread, and asserts that only b1 comes back."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/17-creel-read-sets.md:28
    summary: The Intent rewrites every other stale read-set wording but never says Store.Read's own doc comment changes, and its grep claim omits that hit
    fix: "In card 17's Intent, add a sentence after line 21: Store.Read's doc comment, which says all adds the read messages still retained, names the three sets instead. Reword line 28 so the grep for still kept and still retained is said to hit only the sites this card edits (verbs.go's Store.Read comment, protocol.go's Request.All comment, cli.go's readCmd, doc.go and server_test.go's TestDaemon_Protocol message)."
---

### [MEDIUM] The changed `read --all` protocol assertion is vacuous

**Location:** `_lyx/plan/17-creel-read-sets.md:26`, against `internal/creelengine/server_test.go:293-298`.

**Issue:** In `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread`, alpha holds one message, already acked.
Line 293 already asserts that a plain `read` then returns none.
The card changes line 296's `read --all` assertion from one message to none.
Under the new semantics `--all` returns only unread mail, so on this one-sender pool it returns exactly what plain `read` returns.
The changed assertion repeats line 293, and "once everything is read, returns nothing" is also what the rewritten `TestStore_Mail` `Read_SetsAndInbox` row pins.
Today, line 296 is the only test showing that `Request.All` survives JSON over the socket and reaches `Store.Read` in `server.go:380`, because it expects 1 where plain `read` gives 0.
After the change, nothing pins that wiring: a server that dropped `req.All` would pass every test.
`PATTERN-test-economy` makes a redundant assertion a finding.
The fix should replace it with a discriminating assertion rather than delete it, so the wiring keeps its coverage.

### [LOW] `read --all --from` is prescribed with no named test

**Location:** `_lyx/plan/17-creel-read-sets.md:19`, with the row described at line 25.

**Issue:** Line 19 prescribes that `--all` with `--from` narrows to that sender as `--from` alone does.
The current row covers this combination at `internal/creelengine/verbs_test.go:192` (`readBodies(t, strand(beta), batten, true)` → `b1`).
The rewritten row, as line 25 describes it, covers plain `read`, `--from` and `--all`, but not the combination.
That removes the combination's only coverage, while the Intent still prescribes the behavior.

### [LOW] Store.Read's own doc comment is not named among the wording changes

**Location:** `_lyx/plan/17-creel-read-sets.md:21-28`, against `internal/creelengine/verbs.go:130`.

**Issue:** The Intent names each stale description of the read sets: `Request.All`'s comment (`protocol.go:47`), `readCmd`'s `Long` and `--all` help (`cli.go:397`, `cli.go:428`), and `doc.go:75`.
It leaves out `Store.Read`'s own doc comment, `verbs.go:130`: "all adds the read messages still retained".
Line 28's grep claim looks only at tests.
A fixer who reads only the Intent could change the body and leave the comment describing the retired sets.

## Rejected

- Caller sweep: every other caller of `Store.Read` or `VerbRead` reads mail from one sender only, so the change to plain `read` leaves them alone. These are the plain-read rows of `TestStore_Mail`, `deliver_test.go:119`, `server_test.go` lines 289, 303, 314, 331, 334, 590 and 593, and `creelcli/cli_test.go:302-307`. `holderOf` (`deliver_test.go:565`) asserts only on unread messages, as the card says. `server.go:380` is the only production caller and passes `From` and `All` through unchanged.
- Fidelity: the card carries every item F1 names (the `Store.Read` sets with reserve and ack of exactly what is handed out, the `--all` flag help, `doc.go`'s read paragraph, and the `TestStore_Mail` rows including the two-sender case). Its extra scope, the `Request.All` comment, `readCmd`'s `Long` and the `TestDaemon_Protocol` row, follows directly from F1 and D8.
- Edit targets: `internal/creelengine#Request` with the field change stated in the Intent matches the template rule that a struct field takes no arrow and the Intent states its change. `Store.Read` keeps its signature, so no re-sign arrow applies.
- Granularity: one card for an engine change, its help text, its doc and two test rows is one reviewable unit.
- Verify: `./...` under both run tiers covers `internal/creelengine` and `internal/creelcli`, in the template's order. The `tmux` tier is compiled, not run, which the rubric exempts.
- D20's board-body cut is a step outside the code, and the plan cannot carry it.
