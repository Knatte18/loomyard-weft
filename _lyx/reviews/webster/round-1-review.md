---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/fabricengine/gitexclude.go:96
    summary: ExcludeAnchoredDir's new doc comment is fixed-column wrapped, splitting sentences mid-phrase, against the Comment Line-Break Convention
  - id: F2
    severity: LOW
    location: internal/fabricengine/pathtracked.go:13
    summary: PathTracked's doc comment wraps its first sentence mid-phrase at a fixed column instead of one sentence per line
  - id: F3
    severity: LOW
    location: internal/vscode/config.go:28
    summary: The rewritten WriteConfig doc comment wraps its two new chain sentences at a fixed column, splitting them mid-phrase
  - id: F4
    severity: LOW
    location: internal/fabricengine/gitexclude_integration_test.go:111
    summary: Two new test doc comments (AppendsAnchoredLineOnce, AnchorsToSubpath) are fixed-column wrapped mid-sentence
  - id: F5
    severity: LOW
    location: internal/battenshed/innerrun_ide_test.go:164
    summary: TestInnerRun_OpenErrorIsWarnedNotEscalated counts only WARN lines containing open IDE, so it cannot catch an extra warning the card's exactly-one-warning-line assertion forbids
  - id: F6
    severity: LOW
    location: internal/ideengine/spawn_driven_hub_integration_test.go:96
    summary: The SpawnDriven hub tests identify the attach task by label or task count only, never checking command, args or runOn
  - id: F7
    severity: NIT
    location: internal/battenshed/innerrun.go:140
    summary: Call's new disposition line reads an once-marker; the article should be a
  - id: F8
    severity: NIT
    location: internal/fabricengine/gitexclude_integration_test.go:169
    summary: TestExcludeAnchoredDir_AlreadyIgnoredWritesNothing leaves the .gitignore uncommitted, while the card asks for a committed one
---

Review range: `git diff 0e4610242..HEAD` (`product.parent` is `main`, read from `_lyx/shed/batten-open-ide/status.json`), six card commits over 16 files.
The plan's full `verify:` block passes on HEAD: `go build ./...`, the `go vet` set, the untagged `go test` set, and the `-tags integration` set.

The behaviour matches the cards.
In `Call`, the ide-opened marker is cleared only under `!found`, never on a running re-spawn.
`openIDEOnce` runs after `recordSpawnConfirmed` and before the post-spawn `ReadStatus`.
`callAwaiting` calls it only after a successful re-spawn.
Both failed-spawn paths return before it.
I found no path that opens twice in one run: the marker is written on success and on failure alike, and a fresh child clears it only before its own spawn.
I also found no path that skips the open after the first successful spawn, short of a marker-write failure, which only costs a re-open.

In `SpawnDriven`, `PathTracked` gets `.vscode/tasks.json` for anchor `.` and `wts/some-task/.vscode/tasks.json` for a subpath.
`ExcludeAnchoredDir` gets `/.vscode/` and `/wts/some-task/.vscode/` for the same two anchors.
The tracked branch writes nothing and still launches.
The untracked branch writes the exclude before `WriteConfig`.
`resolveSpawnTarget` logs the prime-resolution warning once, and `Spawn`'s prime-workspace branch keys on `primeResolved && slug == primeName`, which is equivalent to the old `primeErr == nil` condition.

`ExcludeAnchoredDir`'s exit-code handling matches `gitexec.Run`'s contract: a non-zero exit comes back as `*GitError` with `ExitCode` set, and a spawn failure comes back as a plain error that `errors.As` rejects.
The rewrite keeps every line and adds a missing final newline before appending.
The git call goes through the checked `gitexec.Run`, and `cmd/lyx`'s checked-call test passes.

Per-card mechanical check: all six cards are Create, Edit or Prosa.
The Create existence obligation holds.
`PathTracked` is index-only, while `entryMatchesWeft` (`weftgit.go`) also matches `--others` and `detectWarpPollution` (`status.go`) scans a structural pathspec set.
`ExcludeAnchoredDir` has no counterpart among `seedGitExclude`, `unseedGitExclude` or `seedWeftArtifactExcludes`, none of which writes an anchored real-directory entry or short-circuits on `check-ignore`.

Doc comments naming other symbols: `docs/code-comment-conventions.md` restates a rule barring a doc comment from naming other symbols.
`CONSTRAINTS.md` does not encode that rule.
The surrounding code names neighbouring symbols throughout, for example the `Now` field in `deps.go` names `Sleep` and `waitOrCancel`, and `Spawn`'s doc comment names `vscode.WriteConfig`.
The cards also prescribe some of the references the new comments carry.
Verdict: not a defect in this diff, so no finding.

### [LOW] ExcludeAnchoredDir doc comment is fixed-column wrapped

**Location:** internal/fabricengine/gitexclude.go:96
**Issue:** `CONSTRAINTS.md`'s Comment Line-Break Convention requires semantic line breaks, one sentence per line, on every new or changed Go comment.
The new doc comment wraps near column 100, so the first sentence spans three lines ("…anchorRel of the / repository checked out at worktreeDir, and returns the exclude file's resolved path and whether it / wrote anything."), and the later sentences split mid-phrase too.
**Fix:** Rewrite the comment with one sentence per line, breaking further only at independent-clause boundaries.

### [LOW] PathTracked doc comment wraps mid-sentence

**Location:** internal/fabricengine/pathtracked.go:13
**Issue:** The first sentence breaks after "is in the index" and continues on the next line, a fixed-column wrap inside a sentence, against the same convention.
**Fix:** Join the first sentence onto one line.

### [LOW] WriteConfig's rewritten doc comment wraps its new sentences

**Location:** internal/vscode/config.go:28
**Issue:** The two new sentences describing `TaskChainInteractive` and `TaskChainAttachOnly` split mid-phrase ("…written only if they don't / already exist…", "…but tasks.json is / always written…").
The untouched legacy lines below them are out of scope, since the convention bars rewrapping untouched lines.
**Fix:** Put each new sentence on its own line, with a break at the semicolon in the attach-only sentence.

### [LOW] New test doc comments are fixed-column wrapped

**Location:** internal/fabricengine/gitexclude_integration_test.go:111
**Issue:** `TestExcludeAnchoredDir_AppendsAnchoredLineOnce` (line 111) and `TestExcludeAnchoredDir_AnchorsToSubpath` (line 144) each split a one-sentence doc comment across two lines mid-phrase.
The convention covers every Go comment, test files included.
**Fix:** Join each onto one line.

### [LOW] Open-error test filters warnings before counting them

**Location:** internal/battenshed/innerrun_ide_test.go:164
**Issue:** Card 5 asks that the captured output hold exactly one warning line naming the slug and the error.
The test first keeps only lines containing both `WARN` and `open IDE`, so a second, unrelated warning emitted on the failure path would pass unnoticed.
The count is then exact only over the lines the test already expected.
**Fix:** Count every `WARN` line in the captured output, assert there is exactly one, and assert that it names the slug and the error text.

### [LOW] SpawnDriven hub tests check the attach task by label or count only

**Location:** internal/ideengine/spawn_driven_hub_integration_test.go:96
**Issue:** Card 4 asks that the decoded `tasks.json` hold the single attach task.
`TestSpawnDrivenWritesAttachOnlyAndExcludes` checks only the count and label.
`TestSpawnDrivenOverwritesTasksKeepsSettings` (line 143) checks only the count, so an overwrite with any one-task file would pass.
Neither checks the `reed attach` args or `runOn: folderOpen`, which is what makes the task attach when the folder opens.
**Fix:** Add a shared assertion helper that checks exactly one task with label `reed attach`, args `["reed","attach"]` and `runOptions.runOn` `folderOpen`, and use it in both tests.

### [NIT] "an once-marker"

**Location:** internal/battenshed/innerrun.go:140
**Issue:** "once" starts with a /w/ sound, so the article is "a".
**Fix:** Change "an once-marker" to "a once-marker".

### [NIT] Already-ignored exclude test does not commit its .gitignore

**Location:** internal/fabricengine/gitexclude_integration_test.go:169
**Issue:** Card 2 describes this case as "a committed `.gitignore` already listing `.vscode/`", but the test writes the file without committing it.
`git check-ignore` honours an untracked `.gitignore` too, so the assertion still holds.
The test's setup just differs from the scenario the card names.
**Fix:** Stage and commit the `.gitignore` before calling `ExcludeAnchoredDir`.
