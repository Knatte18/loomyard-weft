# Support log — webster-audit-gaps

## Interview

Autonomous run: no operator answered; every turn below is the producer's own question and auto-pick, grounded in the board entry, 5a8be0274 and the code.

1. Is the task the right thing to build?
   Yes: every item is a defect `cli-surface` batch 06 hit or the landing review of 5a8be0274 left open, and each has a reproduction or a named finding.
   No item was dropped or deferred.
2. Item 1, where does the pathless skip live?
   Only `RecoverSpawnOrAttach` passes uncheckable entries to the contract-evidence functions; the other callers pass suspect paths.
   Auto-pick: skip inside `contractWritesFor`, `splitContractPaths` and `uncheckableReason`, as the board expects, through one predicate on the `<class>: <detail>` form, covering every audit class.
3. Item 2, how far do quotes and globs go?
   The 5a8be0274 design note banned any quote or glob; the board asks that a quoted or globbed argument not halt on its own, while loops and expansions still halt.
   Auto-pick: shell-style tokenising with rules on the dequoted word, `$` and backslash always rejected, unquoted globs only after `--` in git segments, tilde admitted as a positional.
4. Item 2, what is the `--json` rule?
   57243019b makes the global `--json` print help before any hook or check; a local `--json` shadows it, and only `lyx shed status` declares one.
   Auto-pick: admit `--json` as a trailing word after command words, optionally after `--help` or `-h`, and pin the local-`--json` set with a test.
5. Item 3, are the flags unbounded today?
   The forms use exact-match lists plus two prefixes; nothing listed writes.
   Auto-pick: a review plus a rejection row per form, code changes only if the review finds a writing flag.
6. Item 4, what is "merge base"?
   The finding name says `merriambase`; the auto-rebaseline path runs no git merge base.
   Auto-pick: `MerriamBase`; the mapping to Stuck holds already, and the gap is the doubled way forward.
7. Item 5, what is stale in the dead refusal?
   A dead fork record and a dead recovery that committed nothing both read "exhausted"; the exhausted case names no route.
   Auto-pick: four cases mirroring recover-batch's verdicts and refusal.
8. Item 6, which tables are thin?
   The findings named cards of a plan that no longer exists.
   Auto-pick: classifier rows come with D2 to D4; rebaseline rows for the "no card file" and "amended in-flight cards" warning branches; shell-wait rows for the two signals and an awaited shell.
9. Test budget?
   Both packages sit at their rows; rows in existing tables, budget edit only if a table is missing.

## Rejected alternatives

- Make `canonicalPath` treat `ENAMETOOLONG` like `ErrNotExist`.
  Rejected: it would hide a genuine resolution fault on a real path and still run the filesystem on shell text.
- Partition uncheckable entries at the recover-batch call site only.
  Rejected: the board names the two evidence functions, and the skip inside them keeps every present and future caller correct; `uncheckableReason` needed the same rule anyway.
- Admit unquoted globs anywhere a positional is admitted.
  Rejected: a planted file name spelling a writing flag would act before `--`; after `--` git reads only pathspecs.
- Admit `$` inside single quotes as text.
  Rejected: the saving is marginal and the double-quote case differs, so one rule for `$` is simpler and conservative.
- Admit `--json` only after `--help`.
  Rejected: the board names bare `lyx ... --json` and the global flag runs nothing on its own; the local-`--json` pin bounds it.
- Teach the classifier which verbs declare a local `--json`.
  Rejected: it would couple `fabricengine` to the command tree; the pinning test keeps the set deliberate instead.
- Keep `merriamBaseError`'s way forward and strip it in `autoRebaseline`.
  Rejected: string surgery on an error message; callers wording their own step is the pattern the step-keyed refusals already use.
- A new top-level test per item.
  Rejected: PATTERN-test-economy and the budget rows both point at table rows.

## Review rounds

_No rounds yet._

## Question ledger

| Question | Status | Answer |
|---|---|---|
| Does any item deserve deferral or an MVP cut? | resolved, auto-pick | No; all six ship together. |
| Where does the pathless skip live? | resolved, auto-pick | Inside the evidence functions and `uncheckableReason`, one predicate (D1). |
| Which quotes, globs and tildes does the classifier admit? | resolved, auto-pick | Dequoted-word rules, globs only after `--` in git, tilde positionals (D2). |
| What exactly does `--json` admit? | resolved, auto-pick | Trailing `--json` after command words, with or without `--help`; local-`--json` set pinned (D3). |
| Are git reader flags unbounded today? | resolved, auto-pick | No; review plus a rejection row per form (D4). |
| What is "merge base" in item 4? | resolved, auto-pick | `MerriamBase`; the gap is the doubled way forward (D5). |
| How many dead cases does begin-batch's remedy need? | resolved, auto-pick | Four, mirroring recover-batch (D6). |
| Which tables get the thin rows? | resolved, auto-pick | Classifier, auto-rebaseline warning branches, shell-wait signals (item 6 criteria). |
| Where does the local-`--json` pin test go if `cmd/lyx` is contested by `cli-surface`? | open for the plan | Prefer `internal/clihelp`, which owns the global flag. |
| Environment faults met during the interview | none | `lyx board` and every read worked. |
