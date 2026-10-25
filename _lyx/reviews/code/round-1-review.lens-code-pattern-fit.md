```yaml
lens: code-pattern-fit
outcome: reported
```

**Task:** check the diff against PATTERN.md, `pattern/` and CLAUDE.md (lens code-pattern-fit).

## List: rules checked

- **`PATTERN-config-layers`** — held. The four retired keys are dropped only through `retiredLoomKeys` and `yamlengine.DropKeys` (`internal/loomengine/config.go:1063-1070`), with a `<key>: retired, removed` report line. The `merge_fix_attempts` and `merge-in` gate-key renames live in `internal/landingshed/config.go:194` and `internal/gateslot/config.go:393-499`. No other package reads the old keys.
- **`PATTERN-refusal-way-forward`** — held in code.
  - The `decision` escalation Reason names `lyx loom decision add`, `circling accept` and `continue`, then `resume` (`internal/shedadapters/bouncer.go` `escalationReason`).
  - The hard-cap halt names `lyx loom resume` (`internal/shedadapters/bouncerdecision.go:172`).
  - The arbiter refusal ends in `HandBackWayForward` (`internal/arbiterstop/arbiterstop.go:76-82`).
  - I did not read the tests for each path; that is the tests lens's call.
- **`PATTERN-shed-recipe-registry`** — held. `escalate_decisions` and `decisions_escalated` are row config keys read by key, and `internal/loomrecipe/decisionswitches_test.go` pins both on Discussion rows only.
- **`PATTERN-stencil-ownership`** — held. The new `bouncer-fragment-decision.md` is read at call time through `stencilstore.Read`. The lens texts point at `lyx loom peers` without restating its output format.
- **`PATTERN-sole-parsers`** — held. `addedDecisionTitles` calls `discussionparser.ParseAddedDecisions`, and review parsing goes through `burlerengine.ParseReview`.
- **`PATTERN-tolerant-records`** — held. The `reportengine` decoders (`internal/reportengine/archivedrecords.go`) use plain `json.Unmarshal` with no `DisallowUnknownFields`. The decision snapshot is YAML, not a JSON record.
- **`PATTERN-leaf-packages` and seam tests** — held. The seam tests only drop `internal/alignment` and `internal/parentreview` from allowed imports. `reportengine` gains `yaml.v3`, and its seam test allows it.
- **Declared PATTERN changes landed** — held. The five `pattern/` files change only for the `Merge-In` → `Parent-Align` rename and the removed alignment setting. I saw no declared-changes list in the constraints I was given, so there is nothing further to check.
- **Commit messages and departures** — held. The commits are `N: slug` plus the attribution line, with no `Pattern-Departure:` trailers and no departure to judge.
- **Fabric's merge-in left alone** — held. `MergeIn`, `MergeInAt`, `MergeInOnly`, `MergeVerbMergeIn` and `lyx fabric merge-in` are untouched. The remaining `Merge-In` strings are intentional retired-name carry-overs (`internal/loomrecipe/loomrecipe.go:19`, `internal/loomcli/subjects.go:237`, `internal/landingshed/parentalign.go:24`, `internal/gateslot/config.go:393`, recipe comments).
- **CLAUDE.md "docs land in the same commit"** — held except F2 below. `doc.go` files, `docs/overview.md`, `README.md` and the specs were updated.
- **CLAUDE.md retired-name sweep (D2, "No text names parent review")** — broken, see F1 and F3.
- **CLAUDE.md "delete what is unused"** (`scribe:code-quality`) — partly broken, see F2.

## Findings

**F1 — MEDIUM, scope.** `internal/loomcli/cli.go:489`
The `lyx loom` command's `Long` help still gives `lyx loom review approve <slug>` as its example. D2 removes the `lyx loom review` verb tree, so the help advertises a command that no longer exists.
Fix: replace the example with a live verb, for example `lyx loom circling accept <slug>`.

**F2 — LOW, scope.** `internal/shedrun/paths.go:242-252`, `internal/loomcli/wiring.go:82` and `:111`, `internal/shedrun/doc.go:30`
`shedrun.AlignmentDir` and `shedrun.AlignmentRel` outlive the deleted alignment agent. `statusCommitPathspec` still stages the alignment directory, and its comment still says it "carries each alignment agent attempt's record". Their only users are that staging and a test.
Wiring comment at `wiring.go:327` says the parent-review directory "is staged no more", so the alignment directory is treated differently with no stated reason.
Fix: either delete the helpers, the staging, their tests and the `doc.go` line (D1 "alignment agent goes whole"), or reword the comment to say the staging deliberately covers directories an older binary left.

**F3 — LOW, consistency.** `internal/creelengine/doc.go:202`
The doc still says "the parent-review gate posts to the parent itself", and that gate no longer exists. The line sits under the future `creel-agents` stage, which D-scope leaves to that note's own rewrite. It is still a reference in package doc to a removed gate.
Fix: reword it to drop the gate, or mark it as the note's historical premise.

**Calls:**
1. `git diff --stat`, the `PATTERN.md` and `pattern/` diff, and a leftover-name grep (the manifest lacked the PATTERN diff).
2. `archivedrecords.go` head and the seam-test diffs (to check tolerant records and import sets).
3. A second leftover grep with `landingshed` excluded (the first was cut off by `head`).
4. Context of `cli.go:489`, `wiring.go:82`, `shedrun/paths.go:242` and the `AlignmentDir` callers (to confirm F1 and F2).
