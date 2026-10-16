---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: ORCHESTRATOR.md:14
    summary: The parent moved the #673 workaround from CLAUDE.md into ORCHESTRATOR.md; the task removed it from CLAUDE.md only, so the merged tree keeps the workaround and the retired anchor name
    fix: Delete the `## Workarounds` section of ORCHESTRATOR.md (the heading and its #673 bullet with the `--anchor own-window` instruction), since `lyx reed add` now defaults to `own-window` and the workaround is dead guidance that names `below-parent`.
---

### [MEDIUM] Workaround survives in ORCHESTRATOR.md

**Location:** ORCHESTRATOR.md:14
**Issue:** The decision record requires "CLAUDE.md holds no #673 workaround" and that no `.md` file outside the listed exceptions names `below-parent`.
The task deleted the workaround from CLAUDE.md.
The parent (commit 0a1f1615c, "ORCHESTRATOR.md rendered in") added `ORCHESTRATOR.md`, and CLAUDE.md's own rule sends orch-only workarounds there, so the same text now lives in ORCHESTRATOR.md.
The two sides touched different files and merged without a conflict.
The orch is told to pass `--anchor own-window` on every `reed add` without `--parent`, citing a default that no longer exists.
The instruction is harmless at runtime, since `own-window` is accepted, but it misstates the default and violates the decision record's retirement sweep.

## Checked and found sound

- `languageconfig.Load` now returns a struct: the burler wiring takes `.Skills` only, which is all the burler path needs (no `VerifyTiers` consumer in `burlerengine` or its wiring), the webster wiring stores `language.VerifyTiers` for both `beginDeps` and `recoverDeps`, and the loom wiring passes `language.VerifyTiers` on.
- Both standalone `reedUp` closures match the task's `func() error` contract, and the recover-batch call site checks `c.reedUp != nil` with no watcher or `watch` reference left in `internal/webstercli` or `internal/burlercli`.
- `docs/overview.md` rows are consistent about the `own-window` default and the verify tiers; the remaining `below-parent` hits are the refusal tests and retired-key fixtures the decision record exempts.
- Test economy: `wantTiers` assertions in `internal/webstercli/wiring_test.go` pin verify-tier plumbing and the task's tests pin `reedUp` arming, so they do not overlap; the recover-batch `reedUp` failure path is covered by `internal/webstercli/verbs_test.go`.
