```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the committed F1–F5 change set (38c6e5b06) against its siblings and cross-references.

1. **MEDIUM**: `internal/shellguard/doc.go`, the new sentence "It reads nothing after a heredoc closed inside a shell -c script, or after two heredoc operators on one line."
   - The first half claims a wider gap than the code has.
   - A quoted script is one `shellUnitPattern` unit, so top-level commands after the closing quote are still read. `bash -c 'cat <<EOF\nx\nEOF'; sed -i s/a/b/ f` reaches `sed` through the units and the `;` separator. This is read off the pattern, not run.
   - Only the later lines of the script itself are unread, which is what F2 reported ("hides the edits after it" inside the script).
   - The twin in the `rows.go` row comment, "An in-place edit passes after a heredoc closed inside a shell -c script", has the same ambiguity.
   - Suggested wording: "an edit later in the same script, after a heredoc closed inside it".

2. **MEDIUM** (pending, if not already next on your list): `_lym/landing/summary.md` (F6) still describes the old state. The context shows no edit to it.
   - It says "Any line that holds a bare word alone, after optional tabs, closes the body". This is now true only for unlisted delimiter words.
   - Its Change section names `closedHeredocPattern` as a pattern that "matches" and names `afterClosedHeredocScope`. The former is now a function and the latter no longer exists.
   - It says the in-place alternatives are matched "in every existing scope and in this one". After a heredoc they now cover all three scopes.
   - It does not mention shell -c scripts after a heredoc, the remaining passes or the new blast radius.

No other findings. `git grep` finds no leftover reference to `afterClosedHeredocScope`, and both `notWordFrom` call sites match the new signature.

Tool calls:
- One `git grep` for `afterClosedHeredocScope`, `closedHeredocPattern` and `notWordFrom(`: my context did not show whether any reference to the removed or changed identifiers remained elsewhere in the tree.
