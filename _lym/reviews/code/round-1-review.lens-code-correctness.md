```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes with one concrete input.

**Per function or branch: the case I ran, and whether it is correct**

- `discussionparser.insertDecision`, drops block.
  - Targets `[" a.go ", "b.go\t"]` are trimmed and each gets its own line after Rationale, with one blank line before them. Correct.
  - `["  "]`, ``["a`b"]`` and `["a\nb"]` return an error and write nothing. Correct.
- `discussionparser.DroppedCreates`.
  - Empty path and missing file give nil, nil. Correct.
  - A line inside a fence, a line under a non-added H3 and a line after `## Constraints` are all ignored. Correct.
  - When two entries drop the same target, the later title wins. Correct.
  - The round trip with the written line works: ``Drops Create: `a.go` `` with trailing `\r` trimmed matches the regex. Correct.
  - A malformed `### Added after Discussion` heading returns a raw error with no way forward. See F3.
- `runplan.withoutDroppedCreates`.
  - Card 1 with a Create group `[foo.go, bar.go]`, both dropped: the copy's group has nil refs, one warning per target, and the caller's slices are untouched. Correct.
  - Nil `dropped` returns a clone with no change. Correct.
  - Edit, Rename and Delete groups are left alone. Correct, but see F1.
- `runplan.PostBatchChecks`, dropped path.
  - Empty `DecisionRecordPath` reads none. Correct.
  - A `DroppedCreates` error comes back before the deferred warnings are set. Correct, but it carries no way forward (F3).
  - The deferred prepend reaches every later return, error returns included. Correct.
  - `CardNotDoneFor` still takes `in.Cards`, but its findings come only from the checked copy. Correct.
  - `RebindResigns`, binding and drift still read the full cards. I did not trace the downstream handling of a dropped and absent target, because `BindHandles` lies outside my reads.
- `websterengine.postBatchInputs`, plus weaver `checkRecordable` and `checkStoppedBatch`. All pass `geom.DecisionRecordPath`, so record-batch and recover-batch run the same checks. Correct.
- `createTargetWayForward`.
  - uncheckable: fresh restart. Correct.
  - exhausted: exhausted steps. Correct.
  - signatureMismatch: amendment alone. Correct.
  - create-not-done from either verb: the three ways. Correct.
  - rename-not-done: offers `--drops-create`, which cannot clear it. Wrong, F1.
- `verifyGateStuckFailure`.
  - The `Entry` branch with 405 bytes of findings is cut and names the report path. Correct.
  - 400 bytes or fewer names "the full report is". Correct.
  - Empty findings give "failed: ; the full report is …". Readable, and acceptable.
  - The `drift` entry appends the `lym drift amend` alternative after the re-entry step. `amend.go` admits it at a blocked Webster and asks for `lym loom resume` after it. Correct.
  - The `gate.Reason` and unreadable-report branches are unchanged. Correct.
- `verifyGateStuckReason` computes `reportPath` once and threads it through. Correct.
- `roundCheck.namedByAmendment`.
  - `_lym/plan/14-card-slug.md` against "…14-card-slug…" or "…card-slug…" matches. Correct.
  - `1-a.md` (stem shorter than 4) is not matched. Correct.
  - Stems such as `00-overview` → `overview`, or `03-test` → `test`, match any entry text that contains that common word. False admission, F4.
- `roundCheck.newFindings`, webster exemption. `origin: webster` at BLOCKING with `HaltReason` set skips the file rule. With no halt reason it is refused, and MEDIUM is refused. Correct as written, but the count is unbounded (F4).
- `roundCheck.carriedFinding`. A finding moved to an amended card passes, even with no changes file. Matches its doc. Correct.
- `priorFindingsBlock`, amending marks.
  - An amending entry among `AddedTexts` is marked and the paragraph is written. Correct.
  - An amending entry added before the previous snapshot, with `AddedTexts` empty: an early return drops the paragraph and the entry. The reviewer never sees it, yet the round check still exempts its cards. Wrong, F2.
- `priorRoundsBlock` and `haltReasonBlock`. An empty reason gives no block. A multi-line reason is quoted line by line with `&gt; `. Correct.
- `shedadapters.amendingEntryTexts`.
  - A nil reader gives nil. A reader error is wrapped and returned. Empty pending gives nil.
  - A digests and texts length mismatch is an error. Titles are filtered in record order.
  - All correct.
- `BuildRoundContext`. Reads the amendment and halt reason. Errors propagate, and the callers turn them into a full round or a parse-only check, as before. Correct.
- `BurlerReviewProducer.Call`, halt read.
  - Empty dir reads nothing.
  - An unreadable file is a producer error, which fails the row and is not logged as absent. Acceptable as fail-loud on a cache file.
  - The probe profile copies `HaltReason`. Correct.
- `WebsterProducer.Call`.
  - Each refusal-error Stuck writes `err.Error()`. Correct.
  - The outcome Stuck writes `claimed.Reason`, which carries the reroute wording. Correct.
  - Done clears the file. Correct.
  - Cancel and engine errors leave an earlier file in place, which a later Done clears. Acceptable.
  - A write or remove failure is logged and the outcome is unchanged, as documented. Correct.
- `runplan.WriteHaltReason`, `ReadHaltReason` and `RemoveHaltReason`: absent, written, removed, and removed again. Correct.
- `loomcli.dropsCreateRefusal`.
  - No targets: allowed.
  - A `--from` file with 2 entries: refused with a way forward.
  - A `--from` file with 1 entry: allowed, and `d.DroppedCreates` is set.
  - `""`, whitespace only, a backtick or a newline: refused.
  - `" a.go "` passes, and `insertDecision` trims it.
  - All correct.
- `renderParentDecisions`. Nil renders `none`. Two entries render `Decision 1:`/`Decision 2:` in order. Correct.
- `burlercli.checkRoundInRunDirectory`. A run directory equal to `&lt;reviews&gt;/plan` reads the amendment and conflict dir. Any other directory reads neither. Correct.
- `shedrecipe.amendmentTitlesAt`. An empty path gives a nil reader. An absent record gives nil titles, nil error. Correct.

**Findings**

1. [MEDIUM] design — `internal/websterengine/recordbatch.go:106` (`createTargetWayForward`)
   - **Summary:** A rename-not-done finding gets the same way forward as create-not-done, including "record `--drops-create &lt;target&gt;` … the done-check skips that target". `withoutDroppedCreates` filters Create groups only, so the parent can follow this route and recover-batch fails again on the same finding.
   - **Evidence:**
     - `internal/runplan/postbatch.go:225-246` skips groups whose type is not `CardTypeCreate`.
     - `recordbatch.go:186-187` passes create-not-done and rename-not-done alike.
     - `recordbatch_test.go` row `RenameNotDoneOffersAmendDropAndRecover` pins the wrong text.

2. [MEDIUM] design — `internal/burlerengine/prompt.go:481-497` (`priorFindingsBlock`)
   - **Summary:** An amending entry that was already in the previous round's snapshot is never shown or marked in the prompt. `BuildRoundContext` puts it in `AmendingTexts`, but the prompt reads only `AddedTexts`.
     - With `AddedTexts` empty, the early return skips the amending paragraph.
     - With other entries added, the paragraph speaks of "marked" entries while none is marked.
   - **Effect:** From the second round after an `--amends-plan` decision on, the reviewer is no longer told which cards are amended or asked to raise them. The round check still exempts those cards silently.
   - **Evidence:**
     - `prompt.go:481-484` returns early.
     - `roundcontext_test.go` row `AmendingTextsIncludeEntriesAlreadyInThePreviousSnapshot` gives AddedTexts=0 and AmendingTexts=1.

3. [LOW] design — `internal/runplan/postbatch.go:282-285`
   - **Summary:** When `DroppedCreates` fails, `PostBatchChecks` returns its raw error. That happens on a malformed `### Added after Discussion` heading, or on a read error. record-batch and recover-batch then return the error unchanged as a refusal with no `way forward:` clause. Every batch now depends on the decision record parsing, and a record that fails stalls all of them with no route named.
   - **Evidence:**
     - `discussionparser/append.go:239-262` returns "line %d: … is not an added-decision heading".
     - `recordbatch.go:577-580` and `recoverbatch.go:729-731` return non-ErrCardNotDone errors unchanged.

4. [LOW] design — `internal/burlerengine/roundcheck.go:332-334` and `:352-375`
   - **Summary:** The exemptions can admit findings they should not, in two ways.
     - **Webster exemption:** any number of new findings tagged `origin: webster` skip the changed-file rule while a halt reason exists. The halt reason persists across every round until Webster ends Done.
     - **Amendment exemption:** `namedByAmendment` matches by plain substring on stems as short as 4 characters. `00-overview` → `overview` and `03-test` → `test` match ordinary words in an entry text, which exempts those files by accident.
   - **Evidence:**
     - `roundcheck.go:332`: `continue` with no count or location tie to the reason.
     - `roundcheck.go:359-372`: `strings.Contains` on the stem and the slug.

Out of my lens: `internal/shedadapters/webster_test.go:290` and `:448` build a `Geometry` without `AnchorRoot`. Any stuck path those tests reach writes `.lym/implementer/webster-halt.md` relative to the package directory. That belongs to the code-tests lens.

**Tool calls**

1. `postbatch.go` 286–330, `recordbatch.go` 570–600 and `recoverbatch.go` 724–745: how the pass continues after the done-checks, and how the verbs surface a non-ErrCardNotDone error. The manifest held only the diff hunks.
2. A grep of `ParseAddedDecisions` callers and `addedSincePreviousSnapshot`: whether `AddedTexts` and `AmendingTexts` come from the same reader, and whether the replace or parse paths drop the new line. Not in the manifest.
3. `ParseAddedDecisions` body and the `decision.go` 1912–1935 and 620–640 callers: whether they handle `Drops Create` lines. Not in the manifest.
4. `prompt.go` 470–500, `runplan.ConflictDir`, and the `Geometry` literals in `webster_test.go`: the no-added early return, and where an empty `AnchorRoot` writes. The diff showed only the changed hunk.
