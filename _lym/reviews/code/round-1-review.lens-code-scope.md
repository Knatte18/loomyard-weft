```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to a Goal point of the board entry, and each Goal point to the changes that build it.

**Decisions (Goal points; the record has no `Added after Discussion` entries)**
- G0, preamble: build items 1 and 2 of #225's Idea. Items 3–6 belong to another task.
- G1 (#227): put the vocabulary rule in the Describe stencil. Built by C2.
- G2 (#227): the description gate refuses a term outside the credit block and names the term and the line. Built by C1.
- G3 (#227): leave the credit block alone, and the check skips it. Built by C1: the check reads only the file, and the block is appended after it.
- G4 (#227): read first which words can match without refusing a repository's own names. This is analysis, not code. C1's narrowed pattern set (no bare task, gate, tier, parent or board) is consistent with it.
- G5 (#225 item 1): a reviewer assistant seat whose findings go back to the writer; it checks terms, claims and deliverables, with the mechanical gate in front. Built by C3.
- G6 (#225 item 2): sync the PR body on every push and verify the two match. Built by C4.
- G7 (#228): the body follows the file on every push, and no round ends while the two differ. C4 builds this only in part (finding F4).
- G8 (#228): after each push, compare and then update or halt. Built by C4.
- G9 (#228): a test that edits the description after the PR exists. Covered by the existing row `AnEditedDescriptionSetsTheTitleAndBodyAgainAtTheNextPollAndOnlyThen` (`prreview_wait_test.go:686`) and the new `ABodyEditedOnGitHub…` row.
- G10 (#244): the elapsed time, the limit and the last state go in the halt reason. Built by C5.
- G11 (#244): decide whether a timeout should be retryable or blocked. Built by C5: Stuck, so the run halts blocked.
- G12 (#241): when a turn end is held after an operator-started turn, send a reminder or a visible notice. Built by no change (finding F2).
- G13 (#241): classify a timeout after an operator interrupt as resumable. Built by C5, which covers every chair timeout.
- G14 (#255 first item): tell the Describe step and any step that writes repository documents not to name lym, LoomYard, gates or other tasks. C2 builds it only in part (finding F3).
- G15 (#255 second item): Code-Review checks documents and the PR body for such names. Built by no change (finding F1).

**Changes and what they trace to**
- **C1, forbidden-term check.** Traces to G2 and G3. Files:
  - `summaryparser/description.go` and its test
  - `landingshed/config.go`, `keynames.go`, `template.yaml`, `config_test.go`
  - the `forbidden` parameter on `NewDescriptionGate` and on `RecordBeamDescription` / `ReplaceBeamDescription`
  - `beamcli/deps.go` and `dobbydeps.go`, `loomcli/validate.go` and `wiring.go` (`DescriptionTerms`), `shedrecipe/entries_gate.go` and `recipe.go`, `envkit.go`
  - the test call sites, `parity_test`, `battencli/wire_seams_test`
- **C2, writer prompts.** `landing-template-describe.md` (vocabulary paragraph) and `darn-template-write.md:9` trace to G1 and part of G14.
- **C3, reviewer seat.** Traces to G5. Files:
  - `describereview.go` and its test, plus the `DescribeTable` changes in `describe.go`
  - `landing-template-describe-review.md`, its `stencils.go` registry entry, and the "Your reviewer" section of `landing-template-describe.md`
  - `loomengine.DescribeReviewDir`, `ReviewDir` wiring in `loomcli/wiring.go`
  - `describeEntry` gate ordering (`entries_describe.go`)
  - the `describe_review` and `describe_review_timeout_min` keys, the `configreg_test` keys, the `shuttleparity/seats.go` holder
- **C4, PR text sync.** Traces to G6, G7, G8 and G9.
  - `pullrequest.Status` Title/Body (`client.go`, `doc.go`, `client_test`) is forced by C4.
  - `prreview.go` `matches`, `syncText` and `wait` signature; `finalize_pr.go` `setRecordedText`; their tests.
- **C5, timeout mapping.** `shedadapters/multillm.go`, `doc.go` and `multillm_test` trace to G10, G11 and G13. Extending the detail to `writerExit` is an extension of G10 to the same adapter, not new scope.
- **C6, docs and specs.** `loom-final-summary-spec.md`, `docs/overview.md`, `landingshed/doc.go` are forced by C1, C3 and C4.

No change lacks a decision: no unrequested scope found.

**Findings**

F1
- severity: MEDIUM · class: scope
- location: `contracts/stencils/loom/loom-rubric-code-review.md`
- summary: #255's second "What to do" item is not built: Code-Review still does not check documents and the PR body for lym or LoomYard names.
- evidence: Goal, `decision-record.md:167`. The change description says so at `_lym/landing/summary.md:61`. The diff's contracts stat shows no change to the rubric, although the rubric ships in this repository (`contracts/stencils/loom/loom-rubric-code-review.md`).
- The entry's own carve-out (`decision-record.md:4`) licenses leaving out only items 3–6 of #225, not #255. No `Added after Discussion` entry records the omission, and the description does not call it a parent correction. So it is an unlicensed gap. "Part of #255" is accurate, but the work is still owed.

F2
- severity: MEDIUM · class: scope
- location: `internal/shuttleengine/hold.go` (held-turn notice for an operator-started turn)
- summary: #241's fix is not built: a held turn end after an operator-started turn with outputs missing still waits silently, with no reminder and no visible notice.
- evidence: Goal, `decision-record.md:146`. The description's own admission is at `summary.md:59-60`. `hold.go:149` (`operatorStartedTurn`) is not in the diff.
- Only the classification half (G13) is built. The entry does not license leaving out the reminder.

F3
- severity: MEDIUM · class: scope
- location: writer stencils for repository documents other than the change description
- summary: #255's first item says "any step that writes repository documents". The diff tells only the Describe writer and the darn writer, and the darn rule covers only the description paragraph, not the repository files the writer changes.
- evidence: Goal, `decision-record.md:166`. `darn-template-write.md:9`: the added sentence sits in the change-description paragraph. No implementer or fixer stencil is in the diff's contracts stat.

F4
- severity: MEDIUM · class: scope
- location: `internal/landingshed/prreview.go:529-534`
- summary: #228 says "refuse to end a round while the two differ". Two paths still end the wait while GitHub holds other text:
  - the continue path skips `syncText` and ends the review;
  - a transient edit failure is swallowed (`syncText` → `pollFailure` returns nil), and `judge` can then return Done on approval.
- evidence: Goal, `decision-record.md:93`. `prreview.go:327-328` and `529-534`.
- `finalize_pr.go` `setRecordedText` corrects the text before a non-beam merge, so the landed text is right. But the round itself ends without the match check the entry asks for.

The Closes and Part-of lines (`summary.md:6-10`) match the built set: #227, #228 and #244 are fully built; #225, #241 and #255 only in part.

Tool calls:
1. Searched `contracts/` for the Code-Review rubric and took the diff stat of `contracts/`, to confirm the rubric ships in this repository and is untouched; the reading manifest held neither.
