```yaml
lens: fix-proposal
outcome: reported
```

The task was to challenge each review proposal the fixer carried out as proposed, building concrete cases where it fails.

**List: proposals carried out as proposed and checked:** F1, F2, F3, F4, F5 (first two refusals), F6 (form 1), F7, F8 (form 2), F9, F10.

**Findings**

1. **MEDIUM.** Concerns F2, D4's "writing each to a sibling temporary file and renaming it into place… a temporary file a crash left behind is ignored".
   - Summary: Record-Write's commit takes the discussion commit, which stages the whole discussion directory. A torn temporary file sitting beside the record therefore gets committed into the weft repository, not ignored.
   - Evidence: `internal/loomcli/wiring.go:322` says "discussionCommitPathspec is the pathspec CommitDiscussion stages: the whole discussion directory". D4 says Record-Write "commits both through the same discussion commit Discussion-Write's output takes".
   - Effect: the "ignored" claim holds for the never-overwrite check but not for the commit. The temporary file needs to sit outside `_lym/discussion/`, or Record-Write needs to remove a leftover one before committing.

2. **MEDIUM.** Concerns F5, D4's "The first two name `lym loom goto --to Record-Write` then `lym loom step`, so Record-Write commits the pair".
   - Summary: for a record that differs from its committed copy later in the run, this route commits an out-of-band hand edit as the record. D6 says "The record changes only through `lym loom decision add`".
   - Second problem: the goto moves a run that is past Darn-Write back to Record-Write. After `step`, the run sits before Drift-Baseline after the writer has committed code. D4's own no-record route exists to avoid exactly that ("a route that never passes Drift-Baseline after the writer committed code").
   - Uncommitted record: this arises only from a crash inside Record-Write. The run's current row is then still Record-Write, so a goto is redundant and `lym loom resume` is enough.
   - Evidence: D4's in-flight bullet with commits; D6's second rationale sentence.
   - Concrete case: the operator edits `decision-record.md` by hand during Code-Review, then runs `decision add`. The refusal sends them to goto Record-Write, Record-Write commits the hand edit, and the run is rewound past Drift-Baseline.

3. **LOW.** Concerns F10, the D10 acceptance row "a run whose current row is Preflight after `lym loom goto --to Preflight`, holding history past Preflight, proceeds".
   - Summary: the row pins no new behaviour. Tapestry-Preflight already waived the goto re-entry, and darn never had the check. The edge the retired guard newly lets through is a status file edited outside those routes, plus the stat race.
   - Evidence: the D10 Bound text "after `lym loom goto --to Preflight`, which Tapestry-Preflight already waived as a deliberate re-entry (`waivesHalfFinished`…)" and "What the dropped checks still caught is a status file edited outside those routes".

4. **LOW.** Concerns F8 form 2, D13's "the orch moves each tuned value into `shed.yaml` or a profile block's `shed` module before landing".
   - Summary: moving a loom-only tuned value into the hub's `shed.yaml` widens it to every recipe, batten and pawl included. That is the widening D13's retire-not-move sentence rejects, so only the profile-block route is consistent.
   - Evidence: D13: "moving them into `shed.yaml` would widen a loom-only value to every recipe". The same "`shed.yaml` or a profile block" wording is repeated in the Open risks line.
   - Side note, outside the fix: `internal/loomengine/config.go:26` says loom's `driver_kind` is the driver "a task's tapestry run is seeded with". Its loss is not bounded, but that predates this fix.

5. **LOW.** Concerns F7's `missingSections` counting only unfenced headings.
   - Summary: a Discussion-written tapestry record with a stray unclosed ``` in an early section now reports every later required section missing, where it passed before. The section check gets stricter for every loom record, not only darn's, and D5's acceptance names only the darn case.
   - Evidence: CommonMark runs an unclosed fence to the end of the document. Today `missingSections` (`validate.go:113-129`) splits every line with no fence tracking.
   - Effect: the result is defensible, but D5's tapestry consequence and its re-prompt cost are unstated.

No case found against F1, F3, F4, F6 or F9. On F3: `migrateShedOverrides` rewrites only values, not key names, so checking before Migrate misses no renamed key, and D13 removes that migration anyway.

Calls:
- grep of `internal/loomengine/config.go` for `driver_kind`/`arbiter`/`tackler`: needed to see whether any Migrate renames a key into a retired one, and what `driver_kind` means. My context lacked Migrate's rename list.
- grep of `internal/loomshed/*.go` for commit and add calls: to find what the discussion commit stages. Not in context.
- grep of `internal/loomcli/wiring.go` for the discussion commit pathspec: the previous grep did not show the staged pathspec.
