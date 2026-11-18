```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal findings, covering the proposals the fixer carried out (F2–F11):

1. **LOW** — `internal/boardcli/intake.go` help text and `plugins/ly/skills/board/SKILL.md` (F6, as proposed).
   - The new wording says the named files "are not on the machine" and tells the reader not to search for them.
   - That holds only once the worktree is gone. The review's own F1 says the "usually gone by import time" premise is unmeasured, and that a named path can still resolve through the provenance slug.
   - An import run while the source worktree still exists tells the reader not to look for files that are there.
   - The import note in `issueContent` says "deleted with its worktree", which is conditional. The help and skill text are now stronger than the note they describe.
   - A softer wording such as "may be gone with their worktree" would hold in both cases.

2. No findings for F2. The reflection stencil is only referenced by `frictionengine/deps.go:61`, `spec.go` and `stencils.go`. Nothing in `frictionengine` pins the "ends with a provenance line" wording. The fixer reworded it to "the last of what you write", with `--attach` sections following.

3. No findings for F3, F5, F7, F8, F9, F10 and F11. I built cases against each and none broke:
   - F5: `utf8.Valid` accepts a BOM and NUL bytes, and refuses UTF-16. That is acceptable because the engine cannot carry those bytes unaltered either.
   - F10: a name made only of backticks yields a delimiter of the run length plus one, padded with spaces, which is a valid span.
   - F11: `"%s"` puts the title in plain quotes, and GitHub titles hold no newline.

4. F4 was fixed differently from the review's proposal: `_` and `*` were kept out of `pathStopChars`.
   - Putting `_` in the stop set, as the review proposed, would truncate real paths such as `.lym/shuttle/startup_capture.txt`.
   - Leading `_` and `*` are accepted instead, and trailing `*` and `_` are trimmed.
   - A path that legitimately ends in `_` loses it, which is negligible.
   - Stopping at `=` and `|` inside a path (`.lym/a=b`) truncates it, which is also negligible.
   - I found no breakage. I traced the new test body `**.lym/a.txt** and |.lym/b|, key=.lym/c and .lym\shuttle\d.txt, foo.lym\e` against the pattern, and it yields the expected four paths.

Further calls made:
- `grep` for `friction-template-reflection` and `provenance line` across `.go`, `.md` and `.yaml`: checked whether any test or code pins the stencil's "ends with a provenance line" wording, which my context lacked.
- `grep -n provenance` in `internal/frictionengine/reflect_test.go`, `spec.go` and `deps.go`: confirmed that only a comment at `deps.go:61` mentions it, with no assertion on the text.
