---
round: 1
focus:
  - "Comment line breaks: CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks on every new or changed Go comment. Check whether the new doc comments on ExcludeAnchoredDir (internal/fabricengine/gitexclude.go), PathTracked (internal/fabricengine/pathtracked.go) and the rewritten WriteConfig (internal/vscode/config.go) break per sentence, or carry fixed-column wraps that split a sentence mid-phrase."
  - "Comment wording: settle whether the new Call disposition line in internal/battenshed/innerrun.go reads correctly ('an once-marker'), and whether the new and changed doc comments in the diff follow the repo's own comment rule in docs/code-comment-conventions.md, which bars naming other symbols. Weigh that rule against CONSTRAINTS.md, which outranks it, and against what the surrounding code already does."
  - "innerrun.go ordering: in Call's spawn branch, check that the ide-opened marker is cleared only for a fresh child (found == false) and never on a running re-spawn. Check that openIDEOnce runs after recordSpawnConfirmed and before the post-spawn ReadStatus, and that callAwaiting calls it only after a successful re-spawn. Settle whether any path can open twice in one run or skip the open after the first successful spawn."
  - "SpawnDriven (internal/ideengine/spawn.go): check the tracked-path query and the exclude entry for both anchors ('.' and a subpath). Confirm the tracked branch writes nothing and still launches, and that the untracked branch writes the exclude before WriteConfig. Confirm the resolveSpawnTarget extraction leaves Spawn's log output and prime-workspace branch unchanged, with exactly one prime-resolution warning."
  - "ExcludeAnchoredDir: check that the check-ignore exit-code handling (0 means ignored, 1 means proceed, anything else is an error) matches gitexec.GitError's real contract. Check that the rewrite keeps every existing line and its final-newline handling, and that the call goes through the checked gitexec.Run, adding no raw site."
  - "Test strength against each card's stated assertions: TestInnerRun_OpenErrorIsWarnedNotEscalated filters warnings to lines containing 'open IDE' but the card asks for exactly one warning line naming the slug and the error. Check the records integration test, the SpawnDriven hub tests (both anchors, the custom exclude line surviving, a clean git status) and the fabricengine exclude and lock tests the same way: do they assert exact content and counts, or only containment?"
  - "Per-card mechanical check: all six cards are Create/Edit/Prosa. Confirm each Create group's 'check nothing equivalent exists first' obligation holds. PathTracked sits next to the existing ls-files scans in fabricengine/status.go and weftgit.go, and ExcludeAnchoredDir next to the existing exclude writers in gitexclude.go. Confirm the plan's verify: commands, including the integration-tagged run, pass on HEAD."
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`: six card commits, 16 files.
`product.parent` is `main`, read from `_lyx/shed/batten-open-ide/status.json`.
No card carries a Rename or Delete group, so per-card mechanical checking reduces to the Create groups' existence obligation plus the plan's `verify:` block.
The focus entries only name questions.
Findings against the plan's own design choices are excluded per the rubric.
One example is the plan checking only `tasks.json` when deciding whether `.vscode/` is tracked.
