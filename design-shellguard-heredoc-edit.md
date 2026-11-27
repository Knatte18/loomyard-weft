# shell-edit guard passes perl -pi / sed -i on a line after a heredoc

`shellguard-heredoc-edit` · Task · bug, process · starting · Worktree-Create

Imported from [issue #268](https://github.com/Knatte18/loomyard/issues/268).

## What happened

A subagent in a lym run edited a markdown file in place with `perl -pi`, and the shell guard let the command run.
A later `sed -i` from the same subagent was refused.

The command had this shape (paths and script text replaced):

```
cd <worktree> && cat > <scratchpad>/fix.pl <<'EOF'
<substitutions>
EOF
f=docs/<file>.md; perl -pi -e "$(cat <scratchpad>/fix.pl)" $f; <read-only checks>
```

## Why it is abnormal

The `shell-edit` row's refusal says `perl -i` is refused, and its `Forms` list `perl -pi`.
The agent that reported this concluded that the `-pi` spelling slips past.
That is probably not the cause: `internal/shellguard/rows_integration_test.go` expects `perl -pi -e s/a/b/ f` to be refused.
The `perl` call here stands after a heredoc operator's line,
and the row's comment in `internal/shellguard/rows.go` says anything after that line passes.
This diagnosis comes from reading the command and that comment; the pattern was not run against the command.

So a documented gap in the guard is reachable by an ordinary pattern: write a script with a heredoc, then run the editor on the next line.
The refusal text gives no hint that the same edit passes in this position.

## What to do

Make the in-place edit match also read the commands that follow a heredoc's closing delimiter, while still skipping the heredoc body.
Add a test row for `sed -i` and `perl -pi` on a line after a closed heredoc.

Provenance: task `no-fake-other-modules`, note `Darn-Write.md`.
