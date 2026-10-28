```yaml
lens: code-affected-set
outcome: reported
```

**Per-change list (lens: code-affected-set)**

| Change | Set found | Covered? |
|---|---|---|
| `MessageLine` quotes the sender (D8) | `reminder.go:36` and `:66`, `cli.go:536`, orch stencil line 39, `orchcli/doc.go:78`, `hookread.go` Long text, `creelengine/doc.go` | Covered, except `tools/sandbox/SANDBOX-CREEL-SUITE.md:115` (F1). |
| PostToolUse and Stop block output replaces `ToolBoundaryContext` | A grep finds no `ToolBoundaryContext` identifier left in Go. `claudeengine/doc.go:127-137`, `capabilities.go` and `creelcli/doc.go:24` are updated. | Covered. |
| `--append-system-prompt-file` replaces `--append-system-prompt` | Go has no leftover use of the old flag. `command.go`, `command_test.go`, `prepare_test.go` and `claudeengine/doc.go:209` are updated. | Covered. |
| Line cap replaces `maxBodyBytes` (D1) | `pool.go:16`, `verbs.go:22`, `creelengine/doc.go:78`, `flags.go` (flag file only for claimable mail) | Covered. |
| Pane path narrowed to non-hook strands (D6, plus the amendments) | `creelengine/doc.go:81, 149, 205, 248`, `docs/overview.md:312, 476-477`, `SANDBOX-CREEL-SUITE.md:5-8` (header note added) | Partly: `SANDBOX-CREEL-SUITE.md` S9 text is not re-checked against the new line shape (F1). |
| New `hook-wait` verb | `client.go` resume case, `PATTERN-daemon-wakeups.md` list, `overview.md`, `creelcli/doc.go` | Covered. |
| Orch stencil and `orchcli/doc.go` rewritten (D11) | Both updated and consistent with D1, D4 and D8. | Covered. |
| Declared PATTERN changes | The diff touches only `pattern/PATTERN-daemon-wakeups.md`. I did not see the directive's declared-changes list, and `PATTERN.md` has no diff. | I can't tell whether any declared change is missing. The handler should check this against the directive's declarations. |
| Commit messages | Every commit is a numbered card title plus the co-author line. I saw no `Pattern-Departure:` trailer. | No departures to judge. |

**Findings**

- **F1 — LOW, consistency.** Location: `tools/sandbox/SANDBOX-CREEL-SUITE.md:115`.
  - Summary: S9 still shows the typed line in the old unquoted shape, `from <shortname>:EXTERNAL:`.
  - Evidence: `MessageLine` now renders `from "<sender>"` (`reminder.go:36`). The same diff edited this file's header (lines 5-8) but left the S9 line stale.
  - Fix: write `from "<shortname>:EXTERNAL":` on line 115.
  - Scope note: the same S9 scenario also says the line is "carried in full" and typed in the pane, and the pane notice path does still exist for hand-added strands. That part is not stale, and S9 stays in scope as a pane-path scenario.

I found no other stale reader. The grep for `from %s:`, `additionalContext` outside `docs/research` and the old flag shows no further Go or stencil use. The `docs/research/` reports are dated measurement records, so I did not flag them.

**Tool calls**
1. `grep` for the old symbols and strings (`ToolBoundaryContext`, `additionalContext`, the old flag, `maxBodyBytes`, the unquoted sender shape). I needed this to find leftover readers beyond the files in the manifest.
2. `grep` for reminder and notice wording in docs, stencils and `doc.go` files. My context lacked those docs outside the files the diff touched.
3. `grep` for hook-related symbols outside `settings.go`, plus `git diff --stat` on `PATTERN.md`, `pattern/` and `CLAUDE.md`. I needed the other readers, and to see which PATTERN files the diff changed.
4. `sed` of S9 in `SANDBOX-CREEL-SUITE.md`, and the diffs of the sandbox doc, the orch stencil and `PATTERN-daemon-wakeups.md`. I had not read those diffs.
5. `grep` for `[creel]` line literals outside tests and `doc.go`. I needed every consumer or producer of the line shape.
