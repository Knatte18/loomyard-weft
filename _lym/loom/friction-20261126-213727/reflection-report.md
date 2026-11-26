# Reflection report

Filed one issue, from `Darn-Write.md`:

- https://github.com/Knatte18/loomyard/issues/268: the `shell-edit` guard passes an in-place edit on a line after a heredoc.

The note blamed the `perl -pi` spelling;
the issue records the heredoc position as the probable cause instead.
