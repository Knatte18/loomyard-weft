```yaml
lens: leanness
outcome: reported
```

## Findings

1. **MEDIUM — Notes for the plan writer: duplicates `relevant.md` and the decisions.** Bullet 1 (pane line order) restates D4's first bullet. Bullets 2–4 list files and members (`Prepare`, `panebin.go`, the five reader files, the record-reader route) that `relevant.md` already carries. The plan writer reads both. Fix: drop the section, or keep only a hint neither file holds.

2. **MEDIUM — Auto-mode assumptions: three of four bullets restate decisions.** "Thresholds per account, no default" is D1; "status line replaces the operator's own" is D5's last paragraph; "account sticky for its lifetime" is D2/D7. Only the `~/.claude` bullet adds anything, and D4's second bullet nearly implies it. Fix: keep a bullet only where it states an assumption the decisions don't already carry, or point to the decision (the section is required by contract, so it may be short).

3. **MEDIUM — D5 rationale: second sentence is a rejected alternative.** "Lym's own transcript totals give tokens… and the OAuth usage endpoint is undocumented…" justifies non-choices; the support log's Rejected alternatives already holds both. Per compaction rules, Decisions carries Decision + Rationale only. Fix: cut that sentence; the "only documented source" sentence and the staleness bound stay (they carry the requirement).

4. **MEDIUM — Constraints: several bullets restate PATTERN entries that bind every task without a task-specific application.** `PATTERN-reed-panes` ("sessions stay interactive") — nothing in the design could violate it. `PATTERN-cli-cobra`, `PATTERN-test-speed`/`PATTERN-test-isolation`, `PATTERN-owner-machine` repeat D2's rationale and D9. The provider-seam, config-layers, lym-files and tolerant-records bullets name task-specific placements and earn their place. Fix: cut the generic bullets.

5. **LOW — Open risks: two bullets restate decision text.** Bullet 1 (figures update only while a session runs; reset times bound it) restates D5's rationale; bullet 4 (second dir lacks instructions/plugins) restates D9 step 1. Fix: keep each fact in one place.

6. **LOW — D3, D7, D8 each end with "the existing usage-limit wait handles it."** Fix: state once in D8; cut from D3 and D7 rationale.

7. **LOW — D9: step-by-step manual procedure is operator setup.** Scope puts it outside the task; it gives the plan writer nothing to build; Acceptance holds it only as a Review check. The board asks that the task state how the second account is wired, so a short statement belongs — the numbered walkthrough could shrink to one sentence naming what the check verifies.

## Other things noticed

8. **LOW — D5 precedence claim may be wrong.** "The worktree's project settings take precedence over user settings" — but `buildLaunchCmd` takes a `settingsPath`, which suggests the engine passes `--settings <file>` rather than writing project `settings.local.json`; the two have different precedence rules. The claim should name the real mechanism, or drop the reason and keep the consequence.

9. **LOW — D2 lists "forks' parents"** among sessions sharing the account — unclear (forks run inside the parent's session, not as separate launches). Fix: drop it.

10. **LOW — D2 pins `.lym/claude/account.json`.** `PATTERN-lym-files` says never-tracked files sit at the mirrored subpath of the `_lym` content they relate to; `_lym/claude/` has no tracked counterpart. Placement detail is plan's; naming "a `lymdirs` path under the anchor's `.lym`" would avoid pinning a path that may not fit the pattern.

Tool calls made: none — the record, support log, rubric and exploration results were in context.
