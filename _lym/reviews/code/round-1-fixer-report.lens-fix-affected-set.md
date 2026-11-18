```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

I reran each set-premise finding's search against the changed tree:

- **F6 (the "evidence not on the board" wording):** the wording is gone from `plugins/ly/skills/board/SKILL.md` and the `intake import` Long help in `internal/boardcli/intake.go`.
  - The remaining "quotes of them" text, in the import note itself and `internal/boardengine/doc.go:454`, matches the review's own judgment that the note's wording is correct.
  - `internal/boardcli/doc.go:188-189` states only the envelope keys, not a claim about evidence.
- **F2 (provenance last):** the only statement of the rule is `contracts/stencils/friction/friction-template-reflection.md:56-57`, now reworded to match. The mention in `internal/frictionengine/deps.go:61` does not conflict.
- **F4 (path detection):** `localPathPattern` is built only in `internal/boardengine/intake.go`, and `pathStopChars` and `pathLeadChars` have no other users.
- **F11 (`%q` on the title):** no other site formats an import title with `%q`.
- **F5 and F10 (attachments):** `appendAttachments` and `attachmentSection` have no callers outside `internal/selfreportcli/cli.go`.

Calls made (all inside the 6-call cap):
- One Bash call grepping the F6, F2, F4, F11 and F5/F10 sets, because the diff and the other sites were not in my context.
- One Bash call reading `internal/boardengine/doc.go:448-458` and grepping the docs for `local_paths`/`empty_body`, because the doc lines were not in my context.
