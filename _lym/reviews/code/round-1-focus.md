---
round: 1
focus:
  - "internal/boardengine/intake.go `importIssue`, `issueContent`: the board entry's first bullet asks to copy named evidence into the entry or a durable file. Does the diff do that, or only add notes and flags? Settle whether the gap against the amended entry is a scope finding."
  - "internal/boardengine/intake.go `importIssue`: `issueContent` is called by the fold and new-note paths, and `localPathsNamed` and `issueBodyEmpty` run again after the write. Settle whether the entry note and the result fields (`EmptyBody`, `LocalPaths`) can disagree, and whether the idempotent already-recorded path returns sensible values."
  - "internal/boardengine/intake.go `localPathPattern` and `localPathsNamed`: check the regexp against realistic issue text (paths in code fences, markdown links, `.lym` mid-component such as `foo.lym/`, Windows separators, trailing punctuation). Settle whether it misses real `.lym/` paths or flags false positives."
  - "internal/selfreportcli/cli.go `appendAttachments`, `attachmentSection`: check the failure paths (unreadable file, non-UTF-8 or binary content, nil body, `--body -` stdin combined with `--attach`, empty attachment) and the rune-limit arithmetic against GitHub's body limit."
  - "internal/selfreportcli and plugins/ly/skills/board/SKILL.md, docs/overview.md, internal/*/doc.go, friction template: confirm the docs the repo's constraints name for a CLI flag and JSON output change land in the diff, and that the prose claims match the code."
  - "Tests in internal/boardengine/intake_test.go, internal/boardcli/intake_test.go, internal/selfreportcli/cli_test.go: check that changed existing tests keep their assertions and still check what their names state; check that new behaviour (fold path flags, attach limit, fence length) has coverage."
  - "Commit messages and summary: check claims against the diff."
---

Seed pass: the answer key is the decision record's board entry alone, so the main scope question is whether flagging satisfies "copy the evidence".
