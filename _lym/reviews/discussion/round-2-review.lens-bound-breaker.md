```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's claims and bounds and tried to break each one.

**Claims attacked**
1. D4 Bound: Shed's reroute cap bounds the send-back edge. Held: `rerouteCount` resets only at PR-Review's own Done or a capped entry (`internal/shedengine/reroute.go:15-31`).
2. D4 Bound: one episode fixes at most `pr_review_small_max` in place, and the count clears "when an episode of its segment begins". **Broken** (F-A).
3. D4: after a send-back, the re-entered PR-Review asks the parent again and starts a new review. **Broken** when the continue flag is already set (F-B).
4. D4: the send-back file in the loom scratch directory is read by the re-entered Code-Bouncer. **Broken** across a machine handover (F-C).
5. D4: the reroute cap "halts for the parent with the cap's way forward". **Broken**: the resume that way forward names never sends back (F-D).
6. D4: the dirty-tree path halts Stuck, and the resumed row meets the mark on a clean tree and reroutes. Held: the mark lives in the tracked pull-request record, and `wake` stops the session and reads the tree again (`internal/landingshed/prreview.go:657-668`).
7. D4: the verb refuses a dirty worktree. Held, assuming the agent writes its findings file under `.scratch/` as its edit directive says.
8. D4: the fix-here fallback catches a perch that approves without review. Held for `zero_rounds`, the sampler entry gate and a missing `on_reroute`. A FullGeneration re-entry forced by the input never resettles.
9. D2: a session adds a trailer later "with no change if need be". **Broken** as a tool fact (F-E).
10. D2 Bound: the regression check section lists every loosened check, "so the pull request body shows every one". **Broken** for trailers committed after Drift-Recheck (F-F).
11. D3: the ruling key is the diff hash plus the range's trailers, and a trailer-only commit triggers a fresh ruling. Held. A moved review base changes the diff hash, so it also triggers a fresh ruling.
12. D3 Bound: only files that exist at the review base are listed, and D1 covers new tests. Held.
13. D5: a task may set `pr_review_small_max` in its `run_settings`. **Broken** against `PATTERN-config-layers` (F-G).
14. D6: the miss is recorded and reaches the operator through a friction note. **Broken** when Tier 2 is off (F-H).
15. D6: one note per send-back, never overwritten. Held: `friction.NotePath` scans for a free name (`internal/friction/friction.go:138-160`).

**Findings**

- **F-A — BLOCKING (on a changed hunk, the F7 fix), design, `_lym/discussion/decision-record.md#D4` (fix-here count clearing, Bound bullet 3)**
  - Summary: PR-Review has no segment in any of the three recipes, and its row does not know when an episode begins, so the fix-here cap has no defined reset point.
  - Counterexample one: if the row clears the count on every `Call` entry, then any re-step clears it and the cap is lost. Re-steps include a transient GitHub or push error, the resume after D4's own dirty-tree Stuck, and a crash-resume. The agent can then fix `pr_review_small_max` more findings after each one.
  - Counterexample two: if the row instead follows Shed's episode rule, a row without a segment ends its episode only at its own Done. The count then carries across a send-back, so the criterion "a new PR-Review episode starts with a fix-here count of zero" never holds after a reroute.
  - Evidence: `contracts/recipes/tapestry-recipe.yaml:523-525` (PR-Review row, no `segment`); `internal/shedengine/run.go:631-703` (`episodeStuckCount`: a standalone producer's episode ends only at its own Done); `internal/landingshed/prreview.go:71-86` (`Call` is re-entered on every re-step).

- **F-B — MEDIUM, design, `_lym/discussion/decision-record.md#D4` (the reroute's clear list)**
  - Summary: the send-back clears the mark, `parent_review_asked` and the session id, but not `Continue` or `MailedHead`.
  - Counterexample: the operator runs `lym loom pr-continue`, which sets `Continue` (`internal/loomcli/prcontinue.go:41`). A gate failure is mailed, or the parent's findings arrive late. The agent runs `pr-findings`, the run sends back, and Code-Review fixes and settles.
  - On re-entry, `wake` gates the new head, and `judge` then takes `rec.Continue && clean && gated` to `endReview` and Done. The run goes to Finalize with no new parent review and no operator turn, which defeats D4's stated re-ask.
  - Evidence: `internal/landingshed/prreview.go:883-904`, `internal/pullrequest/record.go` (`Continue`, `MailedHead`).

- **F-C — MEDIUM, design, `_lym/discussion/decision-record.md#D4` (send-back file stored in the loom scratch directory)**
  - Summary: the file now sits in `.lym/loom`, which a fresh machine starts without. The send-back mark sits in the pull-request record, which is tracked under the run directory.
  - Counterexample: after a send-back, the run is handed over or adopted on another machine. The re-entered Code-Bouncer's `ReviewInput` finds no file, so on an unchanged subject it resettles with no review. Drift-Recheck and Describe run, PR-Review re-enters with the findings dropped, and D6's note names a missing file. The `<n>` that counts send-backs from the directory also restarts.
  - Evidence: `internal/loomengine/config.go:252-258` (`LoomScratchDir` under `.lym`); `internal/shedrun/paths.go:234` (the pull-request record is tracked); `internal/shedadapters/bouncer.go:719-738`.

- **F-D — MEDIUM, design, `_lym/discussion/decision-record.md#D4` (Bound: reroute cap)**
  - Summary: at the cap, the row has already cleared the mark, the session id and the ask before it returns Stuck with `Reroute`. Shed then halts it with the reroute cap's way forward (`internal/shedengine/reroute.go:71`).
  - Counterexample: the parent follows that way forward, which says the resume "grants it 3 more reroutes". The resumed row finds no mark, starts a new session and asks the parent again, so the capped send-back's findings are never routed or fixed.
  - The friction note still records a send-back that never happened. The record states neither this outcome nor a way forward that reaches the findings.
  - Evidence: `internal/shedengine/reroute.go:33-39, 69-76`.

- **F-E — MEDIUM, design (unpinned tool fact), `_lym/discussion/decision-record.md#D2` ("with no change if need be")**
  - Summary: git refuses a commit with no change unless it is given `--allow-empty`. The record does not state that command, and does not pin that the commit paths a session uses accept an empty commit.
  - Those paths are Go-written role commits, Webster's commit wrap and PR-Review's gate-and-push. D3's own way forward, a trailer-only commit, rests on this fact.
  - Evidence: `allow-empty` appears nowhere in `internal/shellguard` or `contracts/stencils` (my grep returned no match).

- **F-F — LOW, consistency, `_lym/discussion/decision-record.md#D2` (Bound bullet 2)**
  - Summary: the claim that the pull request body "shows every one" is broken by a `Loosened-Check:` trailer that a PR-Review fix-here commit adds after Drift-Recheck.
  - `Verdict` renders only the committed drift records, and no Drift row runs after Drift-Recheck, so that trailer never reaches the regression section.
  - Evidence: `internal/driftengine/verdict.go:21-37`; `contracts/recipes/tapestry-recipe.yaml:500-525`.

- **F-G — MEDIUM, design, `_lym/discussion/decision-record.md#D5`**
  - Summary: `run_settings` reach config through the local layer, which Seed-Child rewrites to equal them. A key that can hollow a review gate is a Guard key, and the Guard category closes a key to the local layer (`pattern/PATTERN-config-layers.md`, Categories and Local files).
  - A large `pr_review_small_max` switches D4's send-back off, so it hollows a gate. Either the key is Guard and the stated `run_settings` override is refused, or it is None and any task can disable D4.
  - D5 states neither the category nor the bound. For comparison, `pr_review_ask_parent` is classified none (`internal/loomengine/config.go:673`).

- **F-H — MEDIUM, design, `_lym/discussion/decision-record.md#D6`**
  - Summary: `friction.NotePath` returns "" when the friction directory is empty, which happens whenever Tier 2 is off. With Tier 2 off, D6 writes no note and the miss is recorded nowhere, against the board entry's "so the miss is visible".
  - D6 and its criterion are silent on that case.
  - Evidence: `internal/friction/friction.go:138-142`; `internal/friction/doc.go` (an empty path means off).

Tool calls:
- grep for where the pull-request record lives and who sets `Continue`. My context lacked whether the record is durable (relevant to F-C) and the `pr-continue` writer (relevant to F-B).
