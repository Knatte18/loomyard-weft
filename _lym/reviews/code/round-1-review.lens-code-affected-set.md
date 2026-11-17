```yaml
lens: code-affected-set
outcome: reported
```

**Task:** per change, the set of callers, readers, tests, docs, stencils and PATTERN files outside the touched lines (lens code-affected-set), and whether each is covered.

**Per change: the set found, and whether it is covered**

| Change | Affected set | Covered? |
|---|---|---|
| New `EscalationStalemate` cause | `valid()`, the three cause-list error strings (`circling.go`, `escalation.go`), `arbiterstop.Eligibility`, `loomcli` help and `doc.go`, `orch-role.md`, the arbiter stencil, the escalation brief stencil, `docs/overview.md`. A grep for "four causes", "four legal" and "circling, hard-cap, budget or decision" finds no stale text. Other `EscalationDecision` and `EscalationCircling` sites (`bouncerdecision.go`, `bouncercarryover.go`, `bouncer.go` 1381–1447) act on a specific cause, so they need no stalemate case. | yes |
| Parent notice way forward (`noticeWayForward`) | `renderEscalation`, and `escalationReason` (the awaiting reason), which already names the same verbs and `decision add` for the decision cause. The two texts stay consistent. | yes |
| `review_stalemate_rounds` key | `Config`, `ConfigSpec` as `none`, `LoadConfig` validation, `template.yaml`, `Env` in `shedrecipe/recipe.go`, `loomcli/wiring.go`, `docs/overview.md` (lines 1289–1290 and 1454–1455). `battencli/wire.go` only gets the tree closures. | yes |
| Fixer-report frontmatter | `burler-step-fix.md`, `fix.go` (the two skipped-report constants carry `spawned: false`), the fix gate entry, `ParseFixerReport`, `BuildRoundContext`, `roundUnchanged`. `burler-fix-self-check.md` (the self-check section it asks for follows the changes list) and `burler-template-fixer.md` need no change. | yes |
| Review frontmatter `prior` and `carries` | `ParseCurrentReview` key check, `reviewHeader`, `Finding.Carries`, `burler-review-format.md`, `burler-step-review.md` (its condition matches `priorFindingsHeading`), `ReviewGateEntry` plus `RoundCheckGateEntry` in `reviewTable`. `assist.go:133` gates the assistant's report with `ReviewGateEntry` only, which fits "the entry judges the consolidated review". | yes |
| `validate-review` behaviour | `cli.go` doc comment, `validate.go` help text, `doc.go` in `burlercli`. The old "needs no hub or git repository" wording is gone from the verb, and the remaining grep hits are unrelated files. `docs/overview.md:1632` says only "read-only review-gate self-check", which is still true. | yes |
| Prior-findings block replacing the prior-rounds text | `priorRoundsBlock` and `Profile.Round`. `validateReview` re-roots `WorktreeRoot`. `withParentReportPath` is applied to the full and probe profiles in `Call`. The resume path is not changed (see the lens limit below). | yes, within the lens limit below |
| Fabric verbs `WorktreeTreeID` and `ChangedPathsBetween` | `fabricengine/doc.go`, the `Env` closures, `loomcli/wiring.go` and `battencli/wire.go`. The verbs use `gitexec.RunWithEnv`, not raw git, and are general, so `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary` hold. | yes |
| `lymdirs.ReviewSnapshotDirName` | `reviewTargetSnapshot` names the cache only through `lymdirs`, so `PATTERN-lym-files` holds. | yes |
| Snapshot record `round-<n>-snapshot.json` | Written through `internal/state` with a tracked-file lock, and read by the review producer, fix producer, Bouncer and `BuildRoundContext`. `PATTERN-tolerant-records` holds. | yes |
| Declared PATTERN change (`PATTERN-sole-parsers`) | The diff adds a bullet to the background list and a new section for the review file and fixer-report frontmatter. The entry's own line in `PATTERN.md` is untouched, as the record asks. | landed |
| Commit messages and `Pattern-Departure:` | No trailer in any commit of the range, so there is nothing to judge. | n/a |

**Findings:** none. Every member of the affected set I found is updated or consistent with the diff.

**Lens limit:** I did not trace the resume and probe paths of the fix half beyond the `fixTable` change, which is the code-correctness lens's.

**Tool calls (4)**
1. `git log … | grep Pattern-Departure`, plus a grep for stale cause counts, old key lists and "no git repository" text. My context had no commit messages and no repo-wide text search.
2. `grep -l` for fixer-report, `review_circling_checkpoint` and `spawned` references across contracts, docs and Go. My context lacked the full set of docs and stencils that describe these.
3. Targeted greps in the burler and bouncer stencils, `docs/overview.md` and the call sites of the gate entries. I needed the stencil wording and the other callers of `ReviewGateEntry` and `fixTable`.
4. `sed` of `docs/overview.md` (1628–1640) and `bouncer.go` (1618–1660), plus a grep for the other cause consumers. My context lacked those spans.
