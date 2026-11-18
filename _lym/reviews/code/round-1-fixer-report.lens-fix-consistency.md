```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for consistency with its surroundings (lens fix-consistency).

**Findings**

1. **LOW**, `internal/boardengine/intake.go`, the `ImportResult` doc comment ("Those files are deleted with their worktree, so the entry holds only what the issue quotes of them").
   - F6's fix was to say that an issue can carry named evidence by quoting or attaching it.
   - That change reached `SKILL.md` and the `intake import` help but not this comment, so a twin was left as it was.
   - Evidence: `intake.go` lines 54-55 still read "only what the issue quotes of them".
   - The import note text itself ("only what the issue quotes of them reached the board") also says "quotes" alone.
   - The reviewer called that note correct, since an attachment is body content the issue quotes. The comment is therefore consistent in spirit and only slightly stale.

2. **LOW**, `contracts/stencils/friction/friction-template-reflection.md:56-57`.
   - The rewording from "ends with a provenance line" to "gives a provenance line ... the last of what you write" is coherent with the inline-evidence paragraph.
   - The old rule's twin, the content rule and the "Never file an issue with an empty body" sentence, still agrees.
   - I found no other copy of the stencil: the grep shows only this file and a comment in `frictionengine/deps.go`, which stays accurate.
   - The one residual doubt is that "the last of what you write" is looser than "ends with" for any consumer that parses the final line as provenance.
   - That is a possible oddity rather than a demonstrated defect, so I have no finding beyond this note.

Checked and agreeing:
- `internal/boardcli/doc.go:188-189` describes `empty_body` and `local_paths` without the "not on the board" wording, so it needs no change.
- The `SKILL.md` line and the `intake import` Long help now match each other.
- The `%q` to `"%s"` change in `issueContent` agrees with the updated test rows. The blank-body row and the new fold row both expect the plain-quoted title.
- The help text in `selfreportcli` ("A body the attachments push past GitHub's issue body limit is refused") is consistent with the new UTF-8 refusal, which adds a second refusal without contradicting it. The help doesn't mention the UTF-8 refusal, but it names no exhaustive list.

Tool calls:
1. Grepped the repo for `provenance line` and `ends with a provenance`, to find twins of the stencil rule. My context held only the one file.
2. Grepped for `empty_body`, `local_paths` and `LocalPaths` outside tests, to find docs that describe the same behaviour. My context held only `boardcli/intake.go` and `SKILL.md`.
3. Grepped for `selfreport create` with `attach` or `body` in contracts and plugins, to find guidance on attaching evidence that the change might contradict. It returned nothing.
