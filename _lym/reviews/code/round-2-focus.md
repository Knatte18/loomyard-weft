---
round: 2
focus:
  - "Steer wording: in internal/shellguard/rows.go, check that `editToolsRule`, and every steer built from it (the shell-edit notice and `steerPythonDeny`), names Read for reading a file and grep for searching, and no longer names awk, cat or sed -n as readers. Settle whether the sentences restating it still agree: the row comments in rows.go, internal/shellguard/doc.go, internal/shuttleengine/claudeengine/doc.go, the shuttle entry in docs/overview.md, and internal/shuttleengine/claudeengine/testdata/settings-golden.json."
  - "Edit directive: in contracts/stencils/edit/edit-directive.md and internal/editdirective/doc.go, check that the directive states it overrides harness text permitting shell reads or shell edits in lym sessions, that it no longer lists awk, cat or sed -n, and that the phrases in `editRulePhrases` in contracts/stencils/stencils.go match the directive's wording."
  - "Shell-edit row coverage: in internal/shellguard/rows.go and rows_integration_test.go, check whether the looped `sed -i` spellings (after do or then, behind find -exec, behind xargs with a flag) are either refused or named in the row comment, and whether the `>|` redirect, a quoted redirect target and each `sourceFilePattern` extension have a case. Check internal/shuttleengine/claudeengine/hookcmd_integration_test.go for a shell-edit deny and allow row."
---

The three entries cover the gating wording findings first, then the pattern coverage gaps.
