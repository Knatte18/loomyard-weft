# Fabric audit: plain readers accept unquoted globs and any flag

`audit-reader-globs` · Note · enhancement, webster, fabric

From the webster-audit-gaps PR-Gate review (PR #528, 2026-10-10): the classifier now splits lyx, git and sed segments into shell words and bounds unquoted globs there, but the `readOnlyCommands` readers (`cat *`, `grep -r`) still accept unquoted globs and arbitrary flags, as before the task.
Also, an unquoted glob after `--` is admitted for `git log` and `git diff`; git treats it as a pathspec, but the reason lives only in the code, not in `doc.go`.