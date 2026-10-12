# Allow read-only sed; deny in-place edits; notify on stuck dialogs

Agents spawned by lyx can read with `sed -n` without a refusal, prompt or false audit halt.
Rewriting a file with a script stays banned and is now enforced at the call.
A Claude Code permission prompt that still stalls an autonomous run now reaches the run's parent as a notice.

A Webster implementer fork ran `sed -i` on a test file and `rm -rf "$(cat /tmp/...)"` on a computed path.
Claude Code raised "Dangerous rm operation on statically-unresolvable target", which fires in bypass mode too, and the run waited unseen on the operator.
The edit directive had reached the fork and did not stop it, so the fix denies the calls before they run instead of relying on the wording.

No GitHub issue is closed by this change.

## What changes

- **Edit directive** (`contracts/stencils/edit/edit-directive.md`, summarised in `internal/editdirective/doc.go`): bans `sed -i`, `perl -i`, `awk` and inline-heredoc `python` or `node` by name, each directly after its own "never".
  It says reading with `sed -n`, `grep` or `cat` is fine, and names `.scratch/` in the working directory as the scratch place, removed by literal path.
  `editRulePhrases` matches `sed -i` instead of `sed`, so the single-statement check no longer trips on stencils that mention reading with `sed`.
- **Scratch exclusion** (`internal/fabricengine/gitexclude.go`): fabric adds `/.scratch/` to each wired repo's `.git/info/exclude`.
  Untracked files under the root `.scratch/` no longer show in `git status`, so a dirty-tree check does not see them.
- **Two standing PreToolUse denies** in `claudeengine`'s `settings.go`, installed in every run with no config key:
  - an in-place script edit: `sed` with `-i`, `-i<suffix>`, a cluster holding `i` (`-ni`), or any `--in-place` abbreviation from `--i` up, and `perl` with `i` in a short-flag cluster;
  - `rm` in command position in a command that also holds a command substitution or a shell variable expansion.
- **Python deny steer** now lists `sed -n` among the readers.
- **Fabric-reference classifier** (`readOnlySedSegment` in `internal/fabricengine/refscanner.go`) admits `-n`, `--quiet` and `--silent` with one or more `;`-joined print commands, each addressed by line number or a non-empty `/RE/` with no flag.
  This covers `sed -n '/^func X/,/^}/p' f`, which agents use to read a function and which the audit previously read as a write.
  `s`, `w`, `r`, `e`, a trailing `;`, `//`, a regex flag, `-i`, `-e`, `-s`, `-z`, `$p` and escaped slashes stay non-read-only.
  `websterengine`'s `doc.go` and the `audit: fabric-reference` row of `contracts/specs/refusal-spec.md` state the admitted form.
- **Dialog notice** (`internal/shuttleengine/dialog.go`): an autonomous run on a runner with a notifier latches a permission or input-dialog ask signal.
  The latch is cleared only by a pane read showing an input box, not by a later turn end or turn start.
  Once it is 60 seconds old (`permissionNoticeBound`) and the pane still shows no input box, one notice goes to the run's parent, naming the strand and pointing at `lyx reed attach`.
  The notice's offset is persisted in its own run-state field, so it neither suppresses nor is suppressed by the held-turn-end notice, and a reattach rebuilds the latch only from asks after that offset.
  `QuietHold` and `HoldDeferral` do not defer it.
  Interactive runs, runners without a notifier, and engines without session signals or an input-box reader send nothing.
  The run's classification and outcome are unchanged.
- **`PATTERN-no-denied-recovery`**: `sed -i` and `perl -i` join the closed list and its scan in `cmd/lyx/norecoverydeny_test.go`.

## Design decisions to know

- Both denies are guardrails, not barriers, and say so in their row comments and the `claudeengine` doc.
  They let through `sed` writing by redirection or a `w` command, `awk`, `find -exec sed -i`, anything behind `bash -c`, `eval` or a variable, and a prefix `commandPositionPattern` does not list (`nohup`, `command`, `timeout`).
  The `rm` deny also lets through `find -delete`, `xargs rm` with no `$`, and the special parameters `$$`, `$?`, `$#` and `$!`.
  They falsely deny a matching spelling inside a quoted argument or `grep` alternation, as the python row already does.
- The closed list in `PATTERN-no-denied-recovery` catches only the bare `sed -i` and `perl -i` spellings.
  The other denied spellings and the computed-path `rm` deny are pattern-shaped and stay outside the token scan, like the raw `go` deny; the background file says so.
- No `permissions.allow` rule for `sed` is added: runs use `--dangerously-skip-permissions`, no lyx hook matches `sed`, and a prefix rule such as `Bash(sed -n:*)` would also admit `sed -n 's/a/b/w f'`, a write.
- The dialog notice reads provider-neutral signals only; which Claude Code hook events are ask signals lives in `claudeengine`.
  It goes through the runner's notifier and types nothing into the asking pane.

## To check or run by hand

- The live `llm`-tier scenario in `internal/shuttlecli/smoke_turns_test.go` was not run: it is refused in the build environment and was only vetted with `go vet -tags llm ./internal/shuttlecli`.
  Run it to confirm a fork's prompting command records an ask signal in the run's events file.
  If Claude Code fires neither `PermissionRequest` nor a `permission_prompt` `Notification` for a fork's dialog, the dialog latch needs another trigger.
- The guard branches of the dialog notice in `internal/shuttleengine/dialog.go` (a failed pane read retried on a later tick, a pane read showing a box clearing the latch at any age) may not all be covered by `dialog_test.go`; check each against the tests.
- `internal/shuttlecli/modelpin_test.go` raises `wantSpawnSiteCount` from 14 to 15 for the new scenario's spawn site; confirm that count.
- `internal/fabricengine/dotlyxjunction_integration_test.go` changed to follow the `.scratch/` exclusion; the fabricengine integration run was done with `--tags integration ./internal/fabricengine`, so run the full `lyx gate test --tags integration ./...` before pushing.
- The task `test-tmp-cleanup` plans a sweep of `.scratch/`; it must remove only age-stale entries, or it can delete a live run's scratch files.
- A Claude Code release that changes how the dialog renders could make the input-box read report a box while a dialog shows, silencing the notice.
