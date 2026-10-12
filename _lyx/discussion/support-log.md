# Support log — plan-gate-fixes

## Interview

Autonomous run: no operator answered, so every turn below is the writer's exploration and own pick.

1. Read board entry 426 and the related entries `twill-plan-followups`, `discussion-premise-cites` and `kick-start-pack`; none plans work on `delete-before-reference` or the Plan-Review rubric's fix rules.
2. Read `internal/planglyph/deleteorder.go`: a deleted method is matched by `\.Name\b` over every line of a later card's Edit region, comments and non-Go files included, which explains the `.Run(` and `.Call` floods in #562.
3. Read `callercoverage.go` and `typesload.go`: `caller-uncovered` already resolves method references by receiver type through a slotted `go list` load, falls back to a token scan, and skips a deleted subject's reference inside a later Edit region as `delete-before-reference`'s. Picked: share that computation and partition by region (Decisions 1–2).
4. Found `LaterDeleteReferences`' second caller in `websterengine/recordbatch.go` through `planindex.Index`.
5. Read the Plan-Review rubric and the burler reviewer template: the reviewer's write surface is its review file only, and `validate-plan` takes no plan path. Picked: a stdin overlay on `validate-plan` through `planparser.ParsePlanFrom`.
6. Checked `lyx quarry`: no enclose verb exists, so the enclosing-function directive verifies with `lyx quarry resolve`'s span (Decision 7).
   Board entry `enclose-kickstart` (#180) plans quarry `Enclose` for kick-start packs from tool locations; it does not cover review findings, and once it lands a rubric could point at that verb instead of `resolve`.
7. Lens review, six forks, all returned. Verified the premises findings against `planglyph.go`, `recordbatch.go`, `planparser/validate.go` and `loomcli/validate.go` before acting.
   - premises, BLOCKING: `delete-before-reference` also runs at `ValidateDispatch`, which has no loader — fixed: new Decision 4, scan-only at dispatch and Webster.
   - premises, BLOCKING / scenarios, BLOCKING / pattern-fit, MEDIUM (merged): `checkIndexFileConsistency` lists `plan.Dir` from disk and `EditedDoneCards` reads card bytes from disk, so add, `null` and reorder overlays draw false findings and done-card edits are missed — fixed: the overlay replaces existing files only, every content read goes through its reader, and inserts, removes and renumbers stay out of it (Decisions 5–6).
   - premises, MEDIUM / scenarios, MEDIUM (merged): Webster's batch recording turns every finding into a refusal reason, informational ones included — fixed: it counts blocking findings only (Decision 4, Scope).
   - scenarios, MEDIUM: a type load in Webster's failure path costs a slot wait and may deadlock — fixed: that path runs no load (Decision 4).
   - scenarios, MEDIUM / pattern-fit, LOW / premises, LOW (merged): `--rework` reads committed files the overlay does not reach — fixed: `--overlay` is mutually exclusive with `--rework` and `--require-approved`, which also settles the scenarios LOW on overlaying `approved:`.
   - scenarios, MEDIUM: refusal gaps for empty or terminal stdin, duplicate names, names not on disk, and done cards — fixed: each is refused or specified (Decision 5), with acceptance rows.
   - scenarios, LOW: no nested-module or parse-error case — fixed in part: a parse-error case added; the nested-module case shares the same name-only path and is not given its own row.
   - scenarios, LOW: overlap with an admissible target unspecified — fixed: Decision 1 keeps `delete-before-reference` firing.
   - goal-scope, MEDIUM / board-fit, LOW (merged): which perch rows the overlay directive binds — fixed: it binds writing a finding's `fix`, not a Bouncer focus file (Decision 6).
   - goal-scope, LOW: package-level members and non-Go targets reach beyond the entry — fixed: Scope states the package-level extension and why.
   - goal-scope, LOW: card reorders, inserts and removes not licensed and heavy to overlay — fixed: excluded (Decision 6).
   - goal-scope, LOW: Webster cost unbounded — fixed by Decision 4.
   - goal-scope, LOW: the motivating plan shape has no criterion — fixed: first acceptance row.
   - pattern-fit, LOW: `pattern/PATTERN-gate-self-check-parity.md` mode list — fixed: added to Scope's docs.
   - pattern-fit, LOW: settings must allow a heredoc into `lyx` — fixed: Constraints states it is not a denied in-place edit.
   - board-fit, MEDIUM: overlap with `enclose-kickstart` unnamed — fixed: turn 6 names it; it is no stopgap, since #180 targets kick-start packs, not review findings.
   - board-fit, LOW / leanness, MEDIUM (merged): Decision 6's cost bound undercounts and ignores the Bash timeout — fixed: bound reworded, rubric tells the reviewer to raise the timeout, and Open risks names the effect on `twill-plan-followups`' baseline.
   - leanness, LOW: Auto-mode assumptions and Open risks repeat the Decisions — fixed: both cut to what the Decisions do not say.
   - leanness, LOW: old Decision 4 (own declaration) restated existing behaviour — fixed: dropped, the acceptance row kept.
   - leanness, LOW: `isSubjectObject` aside — fixed: trimmed.
   - leanness, LOW: test tier stated twice — fixed: kept in Constraints and Notes with different content (the rule, and which existing tests flip).
   - leanness, LOW: Webster Out bullet duplicates a decision — fixed: Webster moved into Scope In with its own change.

## Rejected alternatives

- **Keep the textual matcher and only skip comments.** Still flags every `.Run(` on unrelated types, the bulk of #562's findings.
- **A second, independent type load inside `delete-before-reference`.** Doubles the `go list` cost per gate run and lets the two checks disagree on what a reference is.
- **Name-only method matches inside a later Edit region stay blocking.** Reproduces #562 whenever the load fails; `caller-uncovered` already treats the same evidence as informational.
- **Drop name-only matches inside later Edit regions entirely.** Loses the signal with no trace; informational keeps it visible.
- **Thread a loader into `ValidateDispatch` and Webster's batch recording.** Dispatch judges a tree the run has changed, and a slot wait inside Webster's failure path can deadlock against a slot it holds.
- **Widen the reviewer's write surface to a scratch plan copy.** Breaks the reviewer's one-file write rule and needs a plan-path flag anyway.
- **Have the fixer validate instead of the reviewer.** The fixer's gate already does that after applying; #563 asks that the reviewer catch it before proposing.
- **A rubric directive with no tool, asking the reviewer to reason against the spec.** That is today's rule, which failed in `burler-split-budget`.
- **An overlay that adds and removes files, with an overlay-aware directory listing.** Reaches into `planparser`'s orphan check and renumbering for fixes the rubric already routes through `insert-card` and `remove-card`.
- **`--overlay` in every mode.** `--rework` reads committed files the overlay cannot reach, and `--require-approved` would let an overlay pass the approval check.
- **Bump quarry for an `Enclose` verb in this task.** Belongs to `enclose-kickstart`; `resolve`'s span answers the reviewer's question today.

## Review rounds

- Round 1, BLOCKING, every finding fixed:
  - F1, BLOCKING: canonicalization writes card files and re-reads the plan from disk under the overlay — fixed: in-memory substitution and an overlay-reader re-parse (Decision 5), the reviewer's baseline is `--overlay` with `{}` (Decision 6), and an acceptance row.
  - F2, MEDIUM: both delete-before-reference forms at a plan gate with done cards — fixed: Decision 1 names the typed form's entry points and drops the check from the shared resolve-backed pass at the plan gates, with an acceptance row.
  - F3, MEDIUM: `recover-batch`'s refusal undecided — fixed: blocking findings only (Decision 4, Scope), with an acceptance row.
  - F4, LOW: the candidate walk's skipped directories unbounded — fixed: stated in Decision 3's bound.
  - F5, LOW: non-plan `.md` names accepted — fixed: names narrowed to `00-overview.md` and Card Index cards (Decision 5), with refusal rows.
  - F6, LOW: terminal stdin and the `.go`-only import-path match lacked rows — fixed: rows added.
  - F7, LOW: Decision 6's search procedure — fixed: solo runs per fix, the combined run's named cards otherwise, and the bound restated.
  - F8, LOW: no enclosing span — fixed: the finding names the file's self glyph (Decision 7).
- Round 2, APPROVED, every finding fixed:
  - F1, LOW: a joint conflict alongside a solo failure stayed proposed — fixed: every combined-run finding no solo run reproduces withholds the fixes it names (Decision 6), with no extra run.
  - F2, LOW: no plan-gate home for a package self glyph Delete — fixed: its import-path match runs in `planGatePass` at the same entry points and stays blocking (Decision 1).
  - F3, LOW: Decision 5's refusals omitted two the acceptance row requires — fixed: a listed card with no file on disk and the combination with `--rework` or `--require-approved` added (Decision 5).

## Question ledger

- Q: Should a name-only method match inside a later Edit region block? — Resolved, auto-pick: informational, mirroring `caller-uncovered` (Decision 2).
- Q: Does dispatch or Webster's batch recording run a type load? — Resolved, auto-pick: no; scan-only, and Webster counts blocking findings only (Decision 4). Revised after the lens review.
- Q: How does a read-only reviewer validate a proposed fix? — Resolved, auto-pick: `validate-plan --overlay` from stdin (Decision 5).
- Q: Which `validate-plan` modes accept the overlay? — Resolved, auto-pick: the flag-less mode only. Revised after the lens review from "all of them".
- Q: May the overlay add or remove plan files? — Resolved, auto-pick: no, replace existing files only (Decision 5).
- Q: Are non-Go Edit targets still searched? — Resolved, auto-pick: no (Decision 3).
- Q: Does the overlay run's handle canonicalization rewrite or report a non-canonical handle? — Resolved, auto-pick: it substitutes in memory, so later checks see what the Plan-Write gate would leave on disk (Decision 5). Added in review round 1.
- Q: Are later `.go` Edit regions under `testdata`, `vendor`, or `.`- or `_`-prefixed directories searched? — Resolved, auto-pick: no, as `caller-uncovered`'s walk; the go tool never compiles them (Decision 3). Added in review round 1.
- Q: Which run is the reviewer's baseline? — Resolved, auto-pick: `--overlay` with `{}`, which writes nothing (Decision 6). Added in review round 1.
