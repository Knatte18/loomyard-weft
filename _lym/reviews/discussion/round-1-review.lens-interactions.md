```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked each pair of the record's decisions that act on the same state, then checked the acceptance criteria and constraints against them.

**Pairs checked**
- D1 × D2 (fixer record vs `Expired` probe on restart): broken (F1)
- D1 × D8 (store per caller, one record set per gate entry): broken on the restart path, through F1 and F2
- D1 × D4 (crash-attach vs the detached runner's store): broken (F2)
- D1 × D6 (core registers the strand with runproc vs webster "as before"): broken (F5)
- D1 × D3 (caller's timeout vs `fixer_timeout_min` / `recovery_timeout_min`): held
- D2 × D3 (wrap only when the setting holds a model-spec; empty means off): held
- D2 × D5, Darn-Write (an earlier `clean` pass stands; fixer commits and leaves the tree clean; verify fails on a dirty tree): held
- D2 × D5, Plan-Write and Discussion-Write (only one gate entry each, so no earlier pass goes stale): held
- D2 × D7 (writer stops looping only when the resolved entry carries a fixer, the same condition the wrap uses; `Forked` wrapped too): held
- D2 × D4 (a row fixer's own `lym gate test` is a fixer-strand call, keyed by strand name and never handing off): held
- D2's deadline bound × the finalize evaluation: broken (F4)
- D2's attempts bound × a finalize that attaches (one fixer per evaluation, attach counted as that evaluation's fixer): held
- D3 × D4 (`fixer` on the `card` key survives a hub's per-name-map tuning; one timeout): held
- D4 × D6 (a `verify-fixer` strand never hands off; a recovery strand's red card call adds one fixer, stated): held
- D4 × D4 (fixer-phase join ignoring tree identity vs the fixer's own strand-keyed run): held
- D7 × D3 (Webster fork stencil text vs "empty value behaves exactly as today"): broken (F3)
- D7 × D2, Discussion fork boilerplate and lens loop (both inside the fixer-on branch): held
- Constraints × D2 (no `Pending`, blocking closure, `Expired` probe as the only restart path): held
- Acceptance criteria: those for D2 restart (L180), D3 empty value (L181) and D7 Webster fork (L187) inherit F1 and F3. The rest held.

**Findings**

F1 — MEDIUM — design — decision-record.md D2 (L64–65) with D1 (L42) and D8 (L143)
- **Summary:** The `Expired` probe answers true whenever the store holds "a fixer record from before an attach", but no decision says a terminal fixer's record is cleared or marked terminal.
- **Effect:** D8 keeps one record set per gate entry under the run scratch dir. A driver that restarts while the writer is mid-turn, long after an earlier fixer finished, would see that stale record. It would then finalize the run at once, halt gate-failed and throw away the writer's unspent attempts.
- **Fix:** D1 should record or clear the terminal state, and D2's probe should read "a non-terminal fixer record".
- **Evidence:** decision-record.md:42, :64–65, :143. Webster's own attach rule tests the last record's `Status == ""` (internal/websterengine/verifyfixer.go:81).

F2 — MEDIUM — design — decision-record.md D1 (L42) with D4 (L92) and D8 (L144)
- **Summary:** D1 promises that a restarted caller attaches to the live fixer instead of spawning a second. For the agent-call caller, the store is the detached run's own directory.
- **Effect:** A runner that dies mid-fixer (killed, or machine restart) leaves its run unjoinable, so the next call starts a fresh run with a fresh directory. If that run is red it spawns a second fixer in the same worktree while the first is still live. D4's bound covers only "a detached run cancelled on its lapsed lease".
- **Evidence:** decision-record.md:42, :92, :101, :144. `joinable` needs a live runner and `startOrJoin` writes a new run dir (internal/gatecli/detachfront.go:316–327, :572–586, :779).

F3 — MEDIUM — consistency — decision-record.md D7 (L136) against D3 (L81) and the acceptance criteria (L181, L187)
- **Summary:** D7's Webster fork stencil bullet is unconditional: a failed card gate call "has already been through a cold fixer whose reply its report carries".
- **Effect:** D3 and its acceptance criterion say that with the `card` key's `fixer` empty, the caller behaves exactly as today. With the fixer off, the fork prompt would claim a fixer reply that does not exist.
- **Fix:** Render the bullet only when the `card` key's `fixer` is set, as the Plan-Write and Discussion text is.
- **Evidence:** decision-record.md:81, :132–136, :181, :187.

F4 — LOW — consistency — decision-record.md D2 Bound (L68)
- **Summary:** D2 says a finalize evaluation "can outrun the row's deadline by at most one fixer timeout plus one run of the inner gate".
- **Effect:** A deadline or `Expired` finalize runs the wrapped closure from the start: the inner gate, then the fixer, then the inner gate again (D2 steps 1–3). The overrun is two inner runs plus one fixer timeout.
- **Evidence:** decision-record.md:53–56, :68. Finalize evaluates the gate at the deadline (internal/shuttleengine/wait.go:19).

F5 — LOW — consistency — decision-record.md D1 (L36) against D6 (L125) and its acceptance criterion (L175)
- **Summary:** D1 puts runproc registration in the core, so it covers every strand the core starts, webster's `fix-verify` included. D6 and its acceptance criterion say `fix-verify` spawns, waits and records "as before".
- **Effect:** Today webster's cold fixer is not registered with runproc and is stopped only through `RemoveRecoveryStrands`. Once registered, a run's cancel, goto or reset would also end it. That is a webster behaviour change neither decision states.
- **Fix:** Have D1 name registration as part of the core for every caller and D6 state the change, or make registration a caller choice.
- **Evidence:** decision-record.md:36, :125, :175; internal/websterengine/strand.go:134; no runproc call in coldfixer.go or verifyfixer.go.

Tool calls: none. Every point above came from context the manifest already held.
