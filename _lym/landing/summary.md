# shell-edit guard refuses sed -i and perl -pi after a closed heredoc

The shell-edit row's in-place edit match now reads the commands after a heredoc's closing line and still skips the heredoc body.
Before, a command such as

```
cat > .scratch/fix.pl <<'EOF'
s/a/b/;
EOF
f=docs/x.md; perl -pi -e "$(cat .scratch/fix.pl)" $f
```

passed the guard, while `perl -pi` on its own line was refused.

## Cause

`inPlaceEditCommandPattern` reached command position only through `topLevelReachPattern`.
That pattern's run of shell units stops at a heredoc operator and reads only the operator's own line,
so no command after the heredoc's line was ever in command position.

## Change

- `internal/shellguard/rows.go`: `closedHeredocPattern` matches a heredoc operator, the rest of its line, its body lines and a closing line.
  `afterClosedHeredocScope` is the top-level scope read past one or more closed heredocs.
  The in-place edit alternatives, now `inPlaceEditCommandIn`, are matched in every existing scope and in this one.
  The redirect match and every other row are unchanged.
- `internal/shellguard/doc.go` and the shell-edit row's comment state the new reach and what still passes.
- `internal/shellguard/rows_integration_test.go`: rows for `sed -i` and `perl -pi` after a closed heredoc, after a `<<-` tab-indented closing line and after a separator on a later line.
  Other rows pin that `sed -i` inside a heredoc body, inside a body with no closing line, and `sed -n` after a closed heredoc still pass.

## Design decisions

- grep -E has no backreference, so the pattern cannot tie the closing line to the operator's word.
  Any line that holds a bare word alone, after optional tabs, closes the body.
  This is fail-closed: an in-place edit on a later body line after such a line is falsely denied, and the row's comment says so.
- The scope is limited to the in-place edit match, as the entry asks.
  A redirect into a source file after a heredoc's line still passes.
  That gap stays documented in the row's comment.

## Merge danger

**Door:** two-way
**Blast radius:** shellguard
