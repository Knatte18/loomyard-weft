```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, I traced what reads, calls or describes it, and checked whether the diff leaves any of those wrong.

**Per change, the set found and whether it is covered**

1. **`pr-reviewer` role restored** (`agentconfig` roles, template, `stencils.go` opening-stencil map): covered.
   - `roleOpeningStencils`, the shuttleparity producer `buildPRReview`, `TestModules_EveryTemplateKeyClassified` and the `configreg` registration all follow.
   - The stale-wording grep finds no leftover "lym no longer starts" text.
2. **Reinterpretation `agents-pr-reviewer-role-retired` dropped:** covered.
   - `hubreconcile` applies per-module reinterpretations by board marker (`internal/hubreconcile/reinterpret.go:45-54`), so a hub that already applied it keeps a marker nothing reads, which is harmless.
   - No other code names the ID.
3. **`landing.pr_reviewer` is a live key again** (Config, keynames, template, `ConfigSpec`, `LoadConfig`, `config_test`): covered, except one comment.
   - `docs/overview.md:1368` names it in full.
   - The `LoadConfig` doc comment names it bare (finding 1).
4. **`pullrequest.Record` gains `mailed_head` and the session fields:** covered.
   - Readers are the PR-Review row and pr-continue.
   - The record's doc comment and `docs/overview.md` follow.
5. **pr-continue admission, refusal text, help audience and Long text:**
   - Covered: `commitrecords_test` asserts the refusal and admission, the `loomcli` doc.go follows, `PATTERN-landing` follows, `orch-role.md:222-224` and `role-autonomy.md` follow, and `clihelp`'s audience gate reads only the annotation.
   - Not covered: the comments on the `review` mode constant in `beamsetting` and `dobbyengine` (finding 2).
6. **PR-Review session behaviour** (`prreview.go`, `prsession.go`): covered.
   - The `landingshed` doc.go, the `interruptpolicy` comment, the three recipe comments, the `loomengine` retired-keys comment, `docs/overview.md`, the beam template comment and the orch role files follow.
7. **Gate failure mailed instead of halting:**
   - Covered: `docs/overview.md:1373-1374`, the `landingshed` doc.go and the `prreview.go` comments follow.
   - Not covered: those docs and the prompt describe `pr-continue` as a plain retry of the gate, but it also sets the continue flag (finding 3).
8. **The publish guard row comment** (`shellguard/rows.go`): its wording is covered, but the added line is not gofmt-clean (finding 4).
9. **New stencil `landing-template-pr-review.md`:** partly covered.
   - Every verb it names exists.
   - The publish row permits `gh pr comment` and `gh pr review --comment`, and denies approve, request-changes, `git push` and `gh pr edit -B` (rows.go:676-683).
   - Its header comment names the wrong fill function (finding 5).
10. **The `pullRequestDeps` signature gains `runner`:** covered.
    - Every caller is updated: `landingdeps.go`, `landingdeps_integration_test`, `prepush_test` and `watchfilechanges_test`.
    - `wireLanding` fills `SessionLive` and `RegisterSession` for both of its callers (arm.go:528, sharedbootstrap.go:472).
11. **The landingshed import allowlist widened:** covered by the `Loosened-Check:` trailer on 6c9543250. The commit gives a non-geometry reason, so I raise no finding.
12. **The new seams** (`MailAgent`, `Sessions`, `SessionLive`, `RegisterSession`): covered.
    - The battencli and loomcli `intentionallyNil` lists, envkit, `landingdeps_test`'s unfilled list and the `NilSeamIsAnErrorNamingIt` test all list them.
13. **`PATTERN.md` and `pattern/PATTERN-landing.md`:** covered. The index line and background agree with each other and with the code's admission rule. The directive declares no PATTERN changes, so nothing is raised under that bullet.
14. **Commit trailers:**
    - The `Root-Cause:` trailer on 3107202f names the defect the diff removes.
    - The `Loosened-Check:` trailer names its check.
    - No `Pattern-Departure:` trailer is present.

**Findings**

1. **BLOCKING, consistency:** `internal/landingshed/config.go:199`, the `LoadConfig` doc comment.
   - It names `pr_reviewer` bare ("the conflict, describe, pr_reviewer and fixer keys").
   - `PATTERN-config-key-names` requires a config key named in a comment to be written in full, `landing.pr_reviewer`.
   - Evidence: diff piece 011, the hunk at config.go line 196.
2. **LOW, consistency:** `internal/beamsetting/beamsetting.go:35` and `internal/dobbyengine/config.go:39`.
   - Both say the `review` pull request "waits for the operator's own ending of its review".
   - The diff now lets the PR-Review session or an orch end it on the operator's word, and the beam template comment and `PATTERN-landing` were reworded to say so; these two comments were left on the old phrase.
   - Evidence: beamsetting.go:35, dobbyengine/config.go:39.
3. **MEDIUM, design:** `docs/overview.md:1373-1374`, `contracts/stencils/landing/landing-template-pr-review.md` step 3 of "Ending the review", the `prcontinue.go` Long text ("Run again after a gate failure…"), and `gateRetryHint` at `prreview.go:678`.
   - All four present `lym loom pr-continue` as the way to retry a gate failure from outside the change.
   - The verb always sets `Continue` as well (`prcontinue.go`, the `UpdateRecord` call that sets `Continue = true`), so a retry the agent runs because the mail told it to also ends the review once the head gates clean.
   - None of these texts says so, and the mailed hint gives no "only on the operator's word" condition.
   - Evidence: `prreview.go:678,718`; `prcontinue.go` lines 70-73 of the diff; `judge` at `prreview.go:850-854`.
4. **LOW, consistency:** `internal/shellguard/rows.go:1005`.
   - The added comment line is indented one tab short; `gofmt -d` reports it.
   - The repository's rule is that a Go file's formatting is fixed with `lym loom fmt`.
   - Evidence: the `gofmt -d` output for rows.go:1002-1005.
5. **NIT, consistency:** the header comment of `contracts/stencils/landing/landing-template-pr-review.md`.
   - It says "stencil.FillWith requires every marker non-empty".
   - The renderer `RenderPRReviewPrompt` (`prsession.go`) calls `stencil.Fill`, and `FillWith` is a different function with an includes map.
   - Evidence: stencil.go:20,36; `prsession.go`, the `stencil.Fill` call.

**Tool calls**

1. Grepped `contracts`, `docs`, `pattern`, `plugins` and several packages for pr-continue and pr-reviewer: the manifest had no inventory of the docs that name them.
2. Grepped the board mode legend and `hubreconcile`'s reinterpretation handling: needed to judge the dropped reinterpretation ID and any mode legend.
3. Read beamsetting.go:28-42 and the retiring commit's edits to the spec, rubric, `beammodes`, notify, `gotoresume` and `subjects`: needed to find text that commit made which this diff may leave stale.
4. Grepped for leftover "no agent reviews" and "operator's own ending" wording: needed to confirm which stale wording survives.
5. Grepped for other files naming pr-continue and for the `clihelp` audience constants: needed to check who reads the verb's audience change.
