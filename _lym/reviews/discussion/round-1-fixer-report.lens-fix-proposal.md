```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out, as the fix-proposal lens.

Proposals checked: F1, F2, F3, F4, F5, F6, F7, F8, F9 and F10.

**1. [MEDIUM] F4: a damaged record is never reported when the session can't be resumed**
- **Changed text:** in D6's entry bullet, "Any other flag is judged as a wake judges it below, once the session is ensured."
- **Problem:** the proposal handles only the clean case before the session start, so the earlier stuck loop comes back for a flag whose record is damaged.
- **The case:**
  1. A flag carrying the current stamp is read on entry, the checkout is clean and the record fails the guard.
  2. The row ensures a session first. The recorded session can't be resumed, so the row is Stuck with Halt naming `lym loom resume`.
  3. Each resume repeats step 2.
  4. The record's restore steps never reach the operator, and the flag never gets judged.
- **Evidence:** D5's "any other start failure is Stuck with Halt naming `lym loom resume`". D6's damaged-record branch is reached only after that. F4's premise is the same kind of loop.
- **Sound change:** on entry, judge the record check before ensuring a session. Defer only the dirty-checkout branch until the session is up, since it needs a session to mail.

**2. [LOW] F2: a relaunch onto the row drops a flag written while halted**
- **Changed text:** in D6's stamp bullet, "each a Done of the row whose `on_done` names it or a `goto` onto the row".
- **The case:** `shedengine.Relaunch` with a target writes `HistoryEntry{Producer: req.Target, Outcome: OutcomeGoto}` (`internal/shedengine/goto.go:249`). A run halted at Sampler-Session with a flag, then relaunched onto Sampler-Session (after a dead loop, for example), gets a higher stamp. The flag is then ignored with no notice. The old Done-count stamp kept it.
- **Why it matters:** D6's Bound promises that a flag accepted while halted ends the row at the next `lym loom resume`. A plain resume still honours it, but the relaunch path is not named.
- **Sound change:** either count a goto entry only when the row it leaves differs from the target, or state in the Bound that a relaunch onto the row drops the flag.

The other proposals held under challenge:
- **F3:** stopping the session before the check and resuming it before the mail matches `prreview.go:446-466`.
- **F6:** Code-Bouncer's Skipped entry routes to Describe.
- **F7:** `on_reroute: Code-Bouncer` is downstream of the row.
- **F9:** a separate steer needs a shell guard row of its own, which is consistent with `rows.go`'s per-row `Steer`.
- **F1, F5, F8, F10:** no case breaks them.

Tool calls:
- `sed`/`grep` of `internal/shedengine/goto.go:195-250` and the `IdleTimeout` wiring in `internal/loomcli`. My context showed only goto.go's two lines that append the goto entry, not what triggers Relaunch's, and none of the watchdog's idle wiring.
