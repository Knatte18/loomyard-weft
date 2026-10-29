```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed, F2 and F3.

- **LOW, `pattern/PATTERN-parent-directive.md:13`, F3 wording.** The line now says the `edit-directive` stencil is "the only statement of the edit directive's rules". Only the edit rule is guarded by a test.
  - `editRulePhrases` (`contracts/stencils/stencils.go:579`) holds three phrases: `` `sed -i` ``, "heredoc" and "edit or write". None of them matches the own-worktree rule at `contracts/stencils/edit/edit-directive.md:9-10`.
  - Line 15 of the PATTERN still says "A test enforces the rule." That sentence is true of the edit rule only. A stencil restating the own-worktree rule would pass `TestStencils_NoEditRuleRestatement`.
  - So the broadened "only statement" claim on line 13 is unenforced, and line 15 now reads as if it covered it. The review's fix wording is no worse than the old text on this point, but the change leaves the pair inconsistent.
- **LOW, `contracts/stencils/parentdirective_test.go:141`, twin left unchanged.** The `//testtiming:keep` comment quotes "the edit-directive stencil is the only statement of the edit rule" as `PATTERN-parent-directive`'s rule. The PATTERN line no longer reads that way, so the quote is now stale. F3 changed the PATTERN side only.
- **F2, no finding.** I tried to build a counter-case, a place where lym does add an instruction file to a repository, and found none.
  - The grep over `internal/` shows lym only lists and counts instruction files (`websterengine.MasterInstructionFiles`, `MerriamBase`).
  - `MerriamBase` reads the files "when present" and an absent file adds nothing. Nothing in the hits writes one, so "lym adds no instruction file to a repository" holds.

Calls made:
1. `grep -rniE "instruction.?file"` over `.go` and `.md` files, to test whether lym ever adds an instruction file (F2). My context held no code search.
2. `grep` for "edit rule" and "its rules' only" in `contracts`, `pattern` and `internal/editdirective`, to find twins of F3's wording. My context lacked them.
3. `sed -n 575,600p contracts/stencils/stencils.go` plus a `grep -i worktree`, to check whether the test guards the own-worktree rule. My context lacked the test's phrase list.
