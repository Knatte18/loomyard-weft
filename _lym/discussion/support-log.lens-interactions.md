```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the pairs of decisions that act on the same state or mechanism.

**Pairs checked**
- D2 × D5 (Darn-Write: the fixer commits, and the clean gate runs before the wrapped verify): **held**. `loomshed.NewVerifyGate` fails on a dirty tree on its own (`verifytree.StatusDirty`), so a dead or non-committing fixer can't pass the row with a dirty tree.
- D2 × Darn-Write's own re-prompt line: **held**. The findings text carries the fixer's reply, and `DarnGateReprompt` only names the findings file.
- D2 × D8 (restart and attach to the record under the row's `.lym`): **held**. A terminal record leads to a fresh spawn on the next failed evaluation, which still fits "one per failed evaluation".
- D2 × D7 (writer with no self-check, then fixer, then re-prompt with what is left): **held**.
- D7 × the lens-fork boilerplate of D7's own Discussion stencil: **broken** (F1).
- D7 × D3 (where "fixer on" is decided: the stencil versus the recipe element): **weak** (F5).
- D1 × D6 (record ownership): **broken** (F2).
- D1 × D6 × D3 (timeout): **broken** (F3).
- D1 × D6 × D4 (strand role): **broken** (F4).
- D3 × D4 (any agent strand calling a key that sets `fixer`): **held**. The darn writer's computed-scope calls name no key, and `batch` and `webster-fix` have no fixer. Cold strands that render card gates are a separate issue (F6).
- D4 × D5, card gate (the fixer doesn't commit, and the same call runs again before the fork commits): **held**.
- D4 × keep-warm and lease (the fork's re-calls renew the lease while the fixer works): **held**.
- D5 plan and discussion × D2 (the fixer edits `_lym`, the row commits after the session; `Forked` is wrapped too): **held**.
- D2 × D2 finalize bound × the acceptance "a finalize evaluation runs the fixer at most once": **held**.
- Acceptance criteria against D1–D8: each criterion matches the decisions it names. One exception: the D1/D6 criterion "records as before" is silent on which record (F2).

**Findings**

**F1 — MEDIUM — D7 vs the Discussion stencil's lens procedure.** With the fixer on, D7 removes the validator loop and runs the lens review right after writing. But the fork boilerplate in the same stencil still tells every fork the files "were written and passed the self-check before you were spawned". Lens-review step 6 also says "Run `lym loom validate-discussion` again until it passes". D7 changes only the self-check part, so it leaves the fork boilerplate false and keeps the lens-review validate loop. Evidence: the "Lens review by forks" section and "Fork boilerplate" in `loom-template-discussion.md`, which is this prompt.

**F2 — MEDIUM — D1 vs D6 (who owns the record).** D1 says the core saves its own record before the wait. D6 keeps webster's `VerifyFixers` state records, which are fabric-committed in `state.json`. D8 places core records only under the row's `.lym` or the detached run dir. For webster this means two records for one strand, or the core writing to a place D8 does not name. The record store should be a seam the caller supplies. Evidence: D1 bullet 5, D6 paragraph 1, D8.

**F3 — MEDIUM — D3 vs D6 (timeout).** D3 says gate.yaml's `fixer_timeout_min` (30) "bounds every gate fixer's wait". Webster's fix-verify waits on `recovery_timeout_min` (60) today, and D6 says its behaviour is unchanged. The record doesn't say which bound webster's fixer uses. Evidence: `FixVerifyAwait` uses `deps.Config.RecoveryTimeoutMin`.

**F4 — LOW — D1 vs D6 / D4 (strand role).** D1 runs every core strand under role `gate-fixer`. Webster's cold fixer runs as `verify-fixer` today, and D6 doesn't say which role it gets. If it becomes `gate-fixer`, its tokencount identity changes. If it stays `verify-fixer`, D4's "never from a `gate-fixer` strand" guard doesn't cover it; that is harmless today only because the `webster-fix` key has no `fixer`. Evidence: `verifyFixerStrandRole = "verify-fixer"`.

**F5 — LOW — D7 vs D3.** D7 renders the writer's self-check from "the row's fixer setting", but D3 turns the fixer on through the recipe element's `fixer` key, which names that setting. If a recipe element lacks `fixer` while the setting is non-empty, the writer gets no self-check and no fixer. That falls back to the plain re-prompt loop, but with no self-check first. The "on" test should be the resolved gate entry's state, not the setting alone.

**F6 — LOW — D4 bound.** Webster's recovery strand renders `card_gates` (`RenderRecoveryPrompt`), so its `--gate card` calls would start a Sonnet `gate-fixer` under a cold Opus recovery strand. That is not recursion, but the bound doesn't mention it. Evidence: `RenderRecoveryPrompt` fills `card_gates`.

**Tool calls**
- grep for `func NewVerifyGate` and its dirty/clean handling (2 calls, the first matched nothing): my context showed only how `NewVerifyGate` is wired, not whether it refuses a dirty tree, which decides the D2 × D5 pair.
