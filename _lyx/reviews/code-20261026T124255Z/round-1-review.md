---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: pattern/PATTERN-refusal-way-forward.md:18
    summary: The background file still lists sed -i and perl -i as denied forms, and line 23 still treats the in-place spellings as denied but pattern-shaped, though the in-place-edit row and its Forms are gone.
    fix: In pattern/PATTERN-refusal-way-forward.md remove `sed -i` and `perl -i` from the closed list of denied forms on line 18, and remove the in-place spellings clause (`-i<suffix>`, `--in-place` and its abbreviations, a short cluster holding `i`) from the computed-path rm bullet on line 23, keeping the computed-path rm and systemctl parts.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shellguard/rows.go:61
    summary: The commandPositionPattern comment says the separator is reached through a run of shell units as unquotedPrefixPattern reads them, but the diff deleted unquotedPrefixPattern.
    fix: In the comment above commandPositionPattern, reword the sentence to "The separator is reached through a run of shell units, and one after a heredoc operator counts only on that operator's line." with no reference to unquotedPrefixPattern.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: consistency
    location: internal/editdirective/editdirective_test.go:111
    summary: The diff adds assertions on the directive's static prose, a required phrase "by script with `sed -i`" and the absence of "never `sed", which the test economy rule treats as a stencil-prose assertion.
    fix: In TestDirective_RendersTheRuleWithoutBannerOrMarkers restore the single "Edit or Write" anchor check and drop the second wanted phrase and the "never `sed" absence check; restore the test's doc comment to the anchor-phrase wording. The single-statement rule stays covered by TestStencils_NoEditRuleRestatement.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: docs/research/codex-feasibility.md:14
    summary: The deny-guardrails row still lists sed -i among the claudeengine deny guardrails to port to Codex, though that row no longer exists.
    fix: In docs/research/codex-feasibility.md line 14 drop `sed -i` from the list in the "Deny guardrails" row.
    origin: lens:code-pattern-fit
---

### [MEDIUM] PATTERN-refusal-way-forward background still names the removed denies

**Location:** pattern/PATTERN-refusal-way-forward.md:18, :23
**Issue:** The decision record's constraints say `sed -i` and `perl -i` leave the no-denied-recovery scan's list with the row, and `cmd/lyx/norecoverydeny_test.go:228` now asserts `sed -i` is not denied. The background file still states them as "the deny set today" and describes the in-place spellings as denied but pattern-shaped, so it contradicts the code and the test.

### [LOW] Comment names the deleted unquotedPrefixPattern

**Location:** internal/shellguard/rows.go:61
**Issue:** The doc comment of `commandPositionPattern` refers to `unquotedPrefixPattern`, which this diff removed with the redirect-write row, so the comment names a symbol that does not exist.

### [LOW] New assertions on the edit directive's static prose

**Location:** internal/editdirective/editdirective_test.go:111
**Issue:** The test now requires the phrase "by script with `sed -i`" and the absence of "never `sed". Both pin the wording of a stencil's static prose rather than a behavior, which the test economy rule flags.

### [LOW] Research doc lists the removed sed -i guardrail

**Location:** docs/research/codex-feasibility.md:14
**Issue:** The table row of deny guardrails to port still names `sed -i`, a guardrail this diff removes. The document is a research note, so the drift is minor.

## Rejected

- Fork code-correctness F2, comment on `TestShellGuardDenies_RenderedCommandsGrepThePayload` no longer fitting: rejected, the `machine-sleep` pattern is built on `commandPositionPattern`, which embeds `'[^']*'`, so the single-quote case is still covered.
- Fork code-tests F2, redundant `heredoc_appended_to_a_project_file` beside `redirect_into_a_project_file`: rejected, the heredoc payload takes the heredoc-operator branch of the shell-unit pattern that the python, raw-go and computed-rm rows keep, so it is not the same input as a plain redirect.
- Fork code-affected-set note on `docs/research/.../report.md:358` quoting the old directive wording: not a finding, a dated research report records what was true then.
- The golden diff (focus item 2): every one of the 128 changed lines removes text naming the in-place or redirect row and none adds it; I read no other hook, notice sentence or ordering change, and the byte-pinned golden tests in `settings_test.go` cover the rest. Not a defect.
- The directive wording (focus item 3): it permits scripted edits, keeps the `lyx loom fmt` and `.scratch/` rules, drops the `Rename` exception and names only shell tools, so it does not steer to the `python` row. The Webster implementer's rename step still names `rename_tools`, which the board entry leaves out of scope; not a defect.
