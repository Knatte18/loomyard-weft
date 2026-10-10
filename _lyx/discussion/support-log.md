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
   A dead recovery that committed nothing reads "exhausted" below the cap; the exhausted case names no route.
   Only a recovery record is ever terminal dead (`distill` yields done or stuck, `failBatch` failed, and only recover-batch's wait calls `Classify`), so no dead fork case exists.
   Auto-pick: cases mirroring recover-batch's refusal order and `RecoveryRetry`, revised in rounds 2 and 3 (D6).
8. Item 6, which tables are thin?
   The findings named cards of a plan that no longer exists.
   Auto-pick: classifier rows come with D2 to D4; rebaseline rows for the "no card file" and "amended in-flight cards" warning branches; shell-wait rows for the two signals and an awaited shell.
9. Test budget?
   Both packages sit at their rows; rows in existing tables, budget edit only if a table is missing.
10. #518, added to D2's scope by the operator through the orch in round 3: what makes a compound with a `lyx` help call halt?
    Its segments already split at `|` and `;`, and `lyx fabric --help | head` is read-only today; `2>&1` and `2>/dev/null` fail the whole-command `>` and `&` scans, and `sed` is no listed reader.
    Auto-pick: drop the two exact stderr-redirection words before the scans, and admit `sed -n` with a line-print script and plain file words only (D2).

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
- Pin the own-`--json` set in `internal/clihelp`.
  Rejected: the real command tree is built in package `main`, which `internal/clihelp` cannot import, so the pin there would cover a synthetic tree only.
- Teach the classifier which verbs declare a local `--json`.
  Rejected: it would couple `fabricengine` to the command tree; the pinning test keeps the set deliberate instead.
- Keep `merriamBaseError`'s way forward and strip it in `autoRebaseline`.
  Rejected: string surgery on an error message; callers wording their own step is the pattern the step-keyed refusals already use.
- Let an unrendered amendment earn a dead recovery a retry at any count.
  Rejected: Merriam's ladder stops stuck on `recovery_retry: false`, so begin-batch would name a route Merriam does not take unless `RecoveryRetry` changed too, which reaches recover-batch's and status's envelopes.
- Narrow D2's bound to bash, or zsh without `EXTENDED_GLOB`, and record the `^` and `#` residuals as a risk.
  Rejected: the agents' Bash tool runs the operator's own shell profile, and quoting `'^main'` or `'x#'`, or writing a range, costs a reader nothing.
- Admit any redirection to `/dev/null` or any descriptor duplication.
  Rejected: `>/dev/null` hides the output the read exists for, and `1>&2` or `3>&1` gains a reader nothing; two exact words cover the forms agents write.
- Add `sed` to the no-word-check readers.
  Rejected: `sed -i`, a `w` or `e` script, and a flag GNU `sed` reads after its file operands each write or execute.
- Keep begin-batch's exhausted refusal flagless and let Merriam read the text.
  Rejected: Master's begin-batch rung lists record-batch, recover-batch and the next batch, and allows no reset, so a text naming reset steps leaves Merriam no rung; the flag reuses the recover-batch rung's stop.
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
| How many dead cases does begin-batch's remedy need? | resolved, round 3 | Four over every terminal status but done: exhausted at the cap without an unrendered amendment first, then the dead-recovery cases following `RecoveryRetry`, then stuck or failed; no dead fork case, since no path produces one (D6). |
| How does Merriam read begin-batch's exhausted case? | resolved, round 3 | The refusal wraps `ErrRecoveryExhausted`, the verb sets `recovery_exhausted: true`, and Master's begin-batch rung stops stuck on it; the reset steps are the operator's (D6). |
| Does an unquoted `#` count as a glob? | resolved, round 3 | Yes; zsh `EXTENDED_GLOB` reads `x#` as zero or more `x`, so it follows the glob rule (D2). |
| What makes #518's compound read-only? | resolved, round 3 | The exact words `2>&1` and `2>/dev/null` are dropped, and `sed -n` with a line-print script joins the readers; loops and `$v` stay rejected (D2). |
| Does an unrendered amendment earn a dead recovery a retry? | resolved, round 2 | No; a dead recovery follows `RecoveryRetry` alone, as Merriam's ladder does, and the amendment only skips the exhausted case (D6). |
| Does a leading `^` count as a glob? | resolved, round 2 | Yes, unquoted; zsh `EXTENDED_GLOB` negates with it, so it follows the glob rule (D2). |
| Which tables get the thin rows? | resolved, auto-pick | Classifier, auto-rebaseline warning branches, shell-wait signals (item 6 criteria). |
| Where does the own-`--json` pin test go? | resolved, round 1 | `cmd/lyx`, against `newRoot()`; `internal/clihelp` cannot import package `main` and sees only a synthetic tree. |
| Environment faults met during the interview | none | `lyx board` and every read worked. |
